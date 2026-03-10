# Modular Monolith

## Table Of Content

- [Modular Monolith](#modular-monolith)
  - [Table Of Content](#table-of-content)
  - [Querying and Performing Transactions Across Multiple Database Schemas in a Modular Monolith](#querying-and-performing-transactions-across-multiple-database-schemas-in-a-modular-monolith)
    - [API](#api)
    - [Duplicate data](#duplicate-data)
    - [Views](#views)
    - [Have a module to handle data aggregation](#have-a-module-to-handle-data-aggregation)
  - [Data Consistency](#data-consistency)
    - [Event publishing](#event-publishing)
    - [Transaction Manager](#transaction-manager)
    - [Conclusion](#conclusion)

## Querying and Performing Transactions Across Multiple Database Schemas in a Modular Monolith

You have all your modules that are independant from one another. You happen to want to do a merge between data from multiple module: how would you do that? You have an example code right under where Stocks, Carriers and Shipments are three separate modules.

```cs
var dashboard = await context.Shipments
    .Include(s => s.Stocks)
    .Include(s => s.Carriers)
    .Where(s => s.CreatedAt.Date == DateTime.UtcNow.Date)
    .Select(x => new
    {
        ShipmentNumber = x.Number,
        CarrierName = x.Carrier.Name,
        StockLevel = x.Stocks.First().Quantity
    })
    .ToListAsync();
```

### API

Have each module have an API and every time you nned a data, call the api from the other modules in order to recover the data. Our class takes as argument our module APIs and does our request.

```cs
internal sealed class GetShipmentDetailsHandler(
    ShipmentsDbContext context,
    ICarrierModuleApi carrierApi,
    IStockModuleApi stockApi)
{
    public async Task<ErrorOr<ShipmentDetailsResponse>> HandleAsync(
        string shipmentNumber,
        CancellationToken cancellationToken)
    {
        // 1. Get shipment from the local database
        var shipment = await context.Shipments
            .Include(s => s.Items)
            .FirstOrDefaultAsync(s => s.Number == shipmentNumber, cancellationToken);

        if (shipment is null)
        {
            return Error.NotFound("ShipmentNotFound", $"Shipment {shipmentNumber} not found");
        }

        // 2. Get carrier details from the Carriers module
        var carrierDetails = await carrierApi.GetCarrierByNameAsync(
            shipment.Carrier, 
            cancellationToken);

        // 3. Get stock levels for each product from the Stocks module
        var stockLevels = new Dictionary<string, int>();
        foreach (var item in shipment.Items)
        {
            var stockResponse = await stockApi.GetStockLevelAsync(item.Product, cancellationToken);
            if (stockResponse.IsSuccess)
            {
                stockLevels[item.Product] = stockResponse.Quantity;
            }
        }

        // 4. Combine all data into the response
        return new ShipmentDetailsResponse
        {
            ShipmentNumber = shipment.Number,
            OrderId = shipment.OrderId,
            Status = shipment.Status.ToString(),
            CreatedAt = shipment.CreatedAt,
            Carrier = carrierDetails,
            Items = shipment.Items.Select(i => new ShipmentItemDetails
            {
                Product = i.Product,
                Quantity = i.Quantity,
                CurrentStockLevel = stockLevels.GetValueOrDefault(i.Product, 0)
            }).ToList()
        };
    }
}
```

Positive is that we keep a separation of modules and since we work with interfaces, we can easily test each component. Problem is that if we want to retrieve N values, we have to do N calls (one for each shipment value).

### Duplicate data

If your data changes slowly, you can duplicate data every so regularly. problems are eventual consistency, data duplication and the whole stuff is just complicated. It is a possibility, I wouldn't recommend it.

### Views

Create database views thaat "peek" at data from other modules.

```sql
CREATE VIEW shipments_report_view AS
SELECT 
    s.Id,
    s.Number,
    s.OrderId,
    s.Status,
    s.CreatedAt,
    c.Name AS CarrierName,
    c.ContactEmail AS CarrierContactEmail,
    c.PhoneNumber AS CarrierPhoneNumber,
    c.IsActive AS CarrierIsActive
FROM Shipments.Shipments s
LEFT JOIN Carriers.Carriers c ON s.Carrier = c.Name
WHERE c.IsActive = 1;
```

```cs
public class ShipmentWithCarrier
{
    public Guid Id { get; set; }
    public string Number { get; set; }
    public string OrderId { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public string CarrierName { get; set; }
    public string CarrierContactEmail { get; set; }
    public string CarrierPhoneNumber { get; set; }
    public bool CarrierIsActive { get; set; }
}

public class ReadModelsDbContext(DbContextOptions<ReadModelsDbContext> options)
    : DbContext(options)
{
    public DbSet<ShipmentWithCarrier> ShipmentsWithCarriers { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        modelBuilder.Entity<ShipmentWithCarrier>()
            .HasNoKey()
            .ToView("shipments_report_view");
    }
}
```

You can now use your view in your module without having to interact with the database directly:

```cs
var shipments = await context.ShipmentsWithCarriers
    .OrderByDescending(s => s.CreatedAt)
    .Take(request.PageSize)
    .ToListAsync(cancellationToken);

return shipments.Select(s => new ShipmentWithCarrierResponse
{
    ShipmentNumber = s.Number,
    OrderId = s.OrderId,
    Status = s.Status,
    CreatedAt = s.CreatedAt,
    CarrierName = s.CarrierName,
    CarrierContactEmail = s.CarrierContactEmail,
    CarrierPhoneNumber = s.CarrierPhoneNumber
}).ToList();
```

Code is simple and you can have fast queries, our views don't have to contain all of our data, just the columns that we need. Problem is that now your modules are linked with one another and if our data changes, we will have to update the code of the views.

### Have a module to handle data aggregation

Called Composite View pattern, this design pattern handles aggregating data from multiple modules (Also sometimes called Backend For Frontend = BFF). The Composite view pattern exposes its own API and we use YARP to handle calls.

```cs
public class ShipmentDashboardService(
    IHttpClientFactory httpClientFactory)
{
    public async Task<ShipmentDashboardResponse> GetDashboardAsync(
        CancellationToken cancellationToken)
    {
        var client = httpClientFactory.CreateClient("ModularMonolith");

        // Query shipments from the Shipments module
        var shipmentsResponse = await client.GetAsync(
            "/api/shipments?pageSize=10", 
            cancellationToken);
        var shipments = await shipmentsResponse.Content
            .ReadFromJsonAsync<List<ShipmentResponse>>(cancellationToken);

        // Query carriers from the Carriers module
        var carriersResponse = await client.GetAsync(
            "/api/carriers", 
            cancellationToken);
        var carriers = await carriersResponse.Content
            .ReadFromJsonAsync<List<CarrierResponse>>(cancellationToken);

        // Query stock levels from the Stocks module
        var stocksResponse = await client.GetAsync(
            "/api/stocks/summary", 
            cancellationToken);
        var stockSummary = await stocksResponse.Content
            .ReadFromJsonAsync<StockSummaryResponse>(cancellationToken);

        // Combine all data into a dashboard view
        return new ShipmentDashboardResponse
        {
            ...
        };
    }
}
```

With this API:

```cs
public class DashboardEndpoint : ICarterModule
{
    public void AddRoutes(IEndpointRouteBuilder app)
    {
        app.MapGet("/api/bff/dashboard", Handle);
    }

    private static async Task<IResult> Handle(
        ShipmentDashboardService dashboardService,
        CancellationToken cancellationToken)
    {
        var dashboard = await dashboardService.GetDashboardAsync(cancellationToken);
        return Results.Ok(dashboard);
    }
}
```

Advantage is that we separate concerns and avoid having to duplicate code in all of our servoices. We can change this service accordingly, scale it, change it... Difficultty is that it is its own service that you have to take care of.

## Data Consistency

We've seen how to handle transactions across multiple database schema. We also need to check that we have data consistency accross domains.

### Event publishing

==> Publish events when a transaction is complete. Gives us eventual consistency.

```cs
internal sealed class CreateShipmentHandler(
    ShipmentsDbContext context,
    IStockModuleApi stockApi,
    IEventPublisher eventPublisher)
    : ICreateShipmentHandler
{
    public async Task<ErrorOr<ShipmentResponse>> HandleAsync(
        CreateShipmentRequest request,
        CancellationToken cancellationToken)
    {
        // 1. Check if the shipment already exists
        // 2. Check stock levels (read-only operation)
        // 3. Save the shipment in the local database
        
        var shipment = new Shipment { ... };

        await context.Shipments.AddAsync(shipment, cancellationToken);

        // 4. Publish an event for other modules to react
        var shipmentCreatedEvent = new ShipmentCreatedEvent(...);
        await eventPublisher.PublishAsync(shipmentCreatedEvent, cancellationToken);
        
        await context.SaveChangesAsync(cancellationToken);

        return shipment.MapToResponse();
    }
}
```

Advantage of this approach is loose coupling, resilience and scalability. Problem is that we have "only" eventual consistency. We laso have to take care of error handling.

### Transaction Manager

Have a module act as the transaction manager. He takes care of security, consistency and everything. This module will know about all our DbContexts in order to give queries to all of them so we create coupling which can be complicated.

### Conclusion

In the first case, we have eventual consistency, in the second we have strong consistency. What should we do when? Depends on the use case and the guarantees we need. For order processing, notifications & system who  don't need instant feedback, we can have eventual consistency. For financial transactions, booking systems and systems where double booking can be a problem, we need something else.
