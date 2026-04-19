# Message Queues

## Table of content

- [Message Queues](#message-queues)
  - [Table of content](#table-of-content)
  - [Presentation](#presentation)
    - [Queues](#queues)
    - [Topics](#topics)
  - [How messages flow](#how-messages-flow)
  - [Azure service bus](#azure-service-bus)
    - [Configuration](#configuration)
    - [Publishing messages](#publishing-messages)
    - [Handling incoming messages](#handling-incoming-messages)
    - [Deployment](#deployment)

<https://antondevtips.com/blog/synchronous-vs-asynchronous-communication-in-microservices>

## Presentation

### Queues

**Queues** implement point-to-point communication. One sender sends a message, and one receiver processes it. Each message is consumed by exactly one consumer.

Use queues when a message should be processed by a single service. For example, sending a stock update command to the Stocks service.

### Topics

**Topics with Subscriptions** implement publish-subscribe communication. One sender publishes a message to a topic, and multiple subscribers can each receive a copy of that message. Each subscription acts as its own virtual queue.

Use topics when multiple services need to react to the same event. For example, when a payment is processed, both the Fraud Detection service and the Notifications service may need to be notified.

## How messages flow

We need Idempotency to avoid message duplications.

Dead letter queue to catch messages that have failed. Duplicate detection and keep all the messages that were generated in one session in the same session.

## Azure service bus

### Configuration

Azure service bus uses Aspire in order to connect to services:

```sh
dotnet add package Aspire.Hosting.Azure.ServiceBus
dotnet add package Aspire.Hosting.PostgreSQL
dotnet add package Aspire.Hosting.Redis
```

In the Program.cs, add the following line:

```cs
builder.AddAzureServiceBusClient("service-bus");
```

You can connect all the files over here, create a AppHost.cs file to wire everything together.

```cs
var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres");
var redis = builder.AddRedis("cache");

var serviceBus = builder.AddAzureServiceBus("service-bus")
    .RunAsEmulator();

serviceBus.AddServiceBusQueue("update-stock");
serviceBus.AddServiceBusQueue("payment-created");
serviceBus.AddServiceBusQueue("fraud-decision");

var paymentRegisteredTopic = serviceBus.AddServiceBusTopic("payment-registered");
paymentRegisteredTopic.AddServiceBusSubscription("fraud-detection");

var paymentProcessedTopic = serviceBus.AddServiceBusTopic("payment-processed");
paymentProcessedTopic.AddServiceBusSubscription("notifications");

var stocksApi = builder.AddProject<Projects.Stocks_Api>("stocks-api")
    .WithReference(postgres)
    .WithReference(serviceBus)
    .WithExternalHttpEndpoints();

builder.AddProject<Projects.Products_Api>("products-api")
    .WithReference(stocksApi)
    .WithReference(postgres)
    .WithReference(redis)
    .WithReference(serviceBus)
    .WithExternalHttpEndpoints();

builder.AddProject<Projects.Payments_Api>("payments-api")
    .WithReference(postgres)
    .WithReference(serviceBus)
    .WithExternalHttpEndpoints();

builder.AddProject<Projects.FraudDetection_Api>("fraud-detection-api")
    .WithReference(serviceBus)
    .WithExternalHttpEndpoints();

builder.AddProject<Projects.Notifications_Api>("notifications-api")
    .WithReference(serviceBus)
    .WithExternalHttpEndpoints();

builder.Build().Run();
```

In the Extensions.cs file, you should add the source to the openTelemetry tracing configuration:

```cs
.WithTracing(tracing => tracing
    .AddAspNetCoreInstrumentation()
    // ... other sources
    .AddSource("Azure.Messaging.ServiceBus"));
```

### Publishing messages

```cs
public class ServiceBusPublisher(ServiceBusClient client) : IServiceBusPublisher, IAsyncDisposable
{
  // Creating new ServiceBusSenders is expensive so we try to reuse them as much as possible.
    private readonly ConcurrentDictionary<string, ServiceBusSender> _senders = new();

    public async Task PublishAsync<T>(string queueOrTopic, T message, CancellationToken cancellationToken = default)
    {
        var sender = _senders.GetOrAdd(queueOrTopic, client.CreateSender);

        var json = JsonSerializer.Serialize(message);
        var serviceBusMessage = new ServiceBusMessage(json)
        {
            ContentType = "application/json",
            MessageId = Guid.NewGuid().ToString()
        };

        await sender.SendMessageAsync(serviceBusMessage, cancellationToken);
    }

    public async ValueTask DisposeAsync()
    {
        foreach (var sender in _senders.Values)
        {
            await sender.DisposeAsync();
        }

        _senders.Clear();
    }
}
```

And in the Program.cs

```cs
services.AddSingleton<IServiceBusPublisher, ServiceBusPublisher>();
```

Now, if the service has enough stock, we can create an event in the service bus.

```cs
app.MapPost("/products/{id:int}/purchase", async (
    int id,
    PurchaseProductRequest request,
    ProductService productService,
    IStocksApiClient stocksApiClient,
    IServiceBusPublisher serviceBusPublisher) =>
{
    var product = await productService.GetByIdAsync(id);
    if (product is null)
    {
        return Results.NotFound($"Product with ID {id} not found.");
    }

    var hasEnoughStock = await stocksApiClient.HasEnoughStockAsync(id, request.Quantity);
    if (!hasEnoughStock)
    {
        return Results.Conflict("Not enough stock for this purchase.");
    }

    var totalPrice = product.Price * request.Quantity;

    // Publish stock update event (async via Service Bus)
    await serviceBusPublisher.PublishAsync("update-stock",
        new UpdateStockEvent(id, -request.Quantity));

    // Publish purchase completed event for payment processing
    await serviceBusPublisher.PublishAsync("payment-created",
        new PurchaseCompletedEvent(
            TransactionId: Guid.NewGuid().ToString(),
            ProductId: product.Id,
            ProductName: product.Name,
            Quantity: request.Quantity,
            TotalPrice: totalPrice,
            UserId: "user-1",
            Timestamp: DateTime.UtcNow));

    return Results.Ok(new PurchaseProductResponse(
        ProductId: product.Id,
        Quantity: request.Quantity,
        TotalPrice: totalPrice));
});
```

### Handling incoming messages

We start by deserializing messages before routing them to the appropriate sender.

```cs
public class ServiceBusQueueConsumerService<TMessage>(
    ServiceBusClient client,
    IServiceScopeFactory scopeFactory,
    ILogger<ServiceBusQueueConsumerService<TMessage>> logger,
    string queueName) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
        {
            AutoCompleteMessages = true,
            MaxConcurrentCalls = 10
        });

        processor.ProcessMessageAsync += async args =>
        {
            try
            {
                var message = JsonSerializer.Deserialize<TMessage>(args.Message.Body.ToString());
                if (message is not null)
                {
                    using var scope = scopeFactory.CreateScope();
                    var handler = scope.ServiceProvider.GetRequiredService<IServiceBusMessageHandler<TMessage>>();
                    await handler.HandleAsync(message, stoppingToken);
                }
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "Error processing message from queue {QueueName}", queueName);
                throw;
            }
        };

        processor.ProcessErrorAsync += args =>
        {
            logger.LogError(args.Exception, "Error in Service Bus processor for queue {QueueName}", queueName);
            return Task.CompletedTask;
        };

        await processor.StartProcessingAsync(stoppingToken);

        // Keep the service alive while the processor is running.
        while (!stoppingToken.IsCancellationRequested) {}

        await processor.StopProcessingAsync(stoppingToken);
        await processor.DisposeAsync();
    }
}
```

And in the Program.cs

```cs
services.AddSingleton<IServiceBusPublisher, ServiceBusPublisher>();

services.AddScoped<IServiceBusMessageHandler<UpdateStockEvent>, UpdateStockMessageHandler>();
services.AddHostedService(sp => new ServiceBusQueueConsumerService<UpdateStockEvent>(
    sp.GetRequiredService<ServiceBusClient>(),
    sp.GetRequiredService<IServiceScopeFactory>(),
    sp.GetRequiredService<ILogger<ServiceBusQueueConsumerService<UpdateStockEvent>>>(),
    "update-stock"));
```

### Deployment

Aspire supports deployment to azure container apps. While in developemnt, it is enough to use RunAsEmulator but when we start to go in production, we need to connect to the real thing.

```cs
if (builder.Environment.IsProduction())
{
    // Existing Azure Service Bus: queues and topics already exist
    var serviceBus = builder.AddConnectionString("service-bus");

    stocksApi.WithReference(serviceBus);
    productsApi.WithReference(serviceBus);
    paymentsApi.WithReference(serviceBus);
    fraudDetectionApi.WithReference(serviceBus);
    notificationsApi.WithReference(serviceBus);
}
```
