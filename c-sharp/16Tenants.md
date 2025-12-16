# Handling multi tenant apps

## Table of content

- [Handling multi tenant apps](#handling-multi-tenant-apps)
  - [Table of content](#table-of-content)
  - [Approach](#approach)
    - [Single Database](#single-database)
    - [Multiple databases](#multiple-databases)

## Approach

If you serve multiple customers, you can either have multiple databases or one database with logical isolation of tenants.

### Single Database

Using Query filters is an approach:

```cs
public class OrdersDbContext : DbContext
{
    private readonly string _tenantId;

    public OrdersDbContext(
        DbContextOptions<OrdersDbContext> options,
        TenantProvider tenantProvider)
        : base(options)
    {
        _tenantId = tenantProvider.TenantId;
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .Entity<Order>
            .HasQueryFilter(o => o.TenantId == _tenantId);
    }
}

```

We know how to filter our clients, now how do we know which is the right tenant?

- Include it in the header of the request.
- JWT claim.
- API Key.

Here is an example with the header:

```cs
public sealed class TenantProvider
{
    private const string TenantIdHeaderName = "X-TenantId";

    private readonly IHttpContextAccessor _httpContextAccessor;

    public TenantProvider(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public string TenantId => _httpContextAccessor
        .HttpContext
        .Request
        .Headers[TenantIdHeaderName];
}
```

### Multiple databases

In this case, we need to store the tenant information and connection strings in the application settings or other.

```json
"Tenants": {
    { "Id": "tenant-1", "ConnectionString": "Host=tenant1.db;Database=tenant1" },
    { "Id": "tenant-2", "ConnectionString": "Host=tenant2.db;Database=tenant2" }
}
```

The TenantProvider class handles setting the connection string and we can create DbContexts with the appropriate connection string.

```cs
public sealed class TenantProvider
{
    private const string TenantIdHeaderName = "X-TenantId";

    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly TenantSettings _tenantSettings;

    public TenantProvider(
        IHttpContextAccessor httpContextAccessor,
        IOptions<TenantSettings> tenantsOptions)
    {
        _httpContextAccessor = httpContextAccessor;
        _tenants = tenantsOptions.Value;
    }

    public string TenantId => _httpContextAccessor
        .HttpContext
        .Request
        .Headers[TenantIdHeaderName];

    public string GetConnectionString()
    {
        return _tenantSettings.Tenants.Single(t => t.Id == TenantId);
    }
}

builder.Services.AddDbContext<OrdersDbContext>((sp, o) =>
{
    var tenantProvider = sp.GetRequiredService<TenantProvider>();

    var connectionString = tenantProvider.GetConnectionString();

    o.UseSqlServer(connectionString);
});
```
