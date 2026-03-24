# Cache

## Table of Content

- [Cache](#cache)
  - [Table of Content](#table-of-content)
  - [ForeWarning](#forewarning)
  - [Example of Memory Cache](#example-of-memory-cache)
  - [HybridCache](#hybridcache)
  - [Cache Aside](#cache-aside)
  - [Read Through cache](#read-through-cache)
    - [Write Around](#write-around)
  - [Write back](#write-back)
  - [Write through](#write-through)
  - [Database result caching](#database-result-caching)
  - [Output Cache](#output-cache)
    - [Basic version](#basic-version)
    - [Additional parameters](#additional-parameters)
    - [Cache eviction](#cache-eviction)
    - [In memory vs in database](#in-memory-vs-in-database)
    - [Risks](#risks)

## ForeWarning

Before you put cache in place, make sure you clarify the cache policy: how long do we keep an item in memory? How many of them do we keep... All of these things need to be discussed beforehand.

## Example of Memory Cache

You have to configure cache in your application with timeout and parameters. You can then put data in cache. When you want to access them, just cache.In the cache, you can then TryGetValue.

You can store cache in multiple places. It can be in the memory of your application, in a distributed cache (ex: redis). There is a third option called HybridCache that does the best of both worlds.

```cs
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);
builder.Services.AddMemoryCache();
using IHost host = builder.Build();

IMemoryCache cache =
    host.Services.GetRequiredService<IMemoryCache>();

const int MillisecondsDelayAfterAdd = 50;
const int MillisecondsAbsoluteExpiration = 750;

static void OnPostEviction(
    object key, object? letter, EvictionReason reason, object? state)
{
    if (letter is AlphabetLetter alphabetLetter)
    {
        Console.WriteLine($"{alphabetLetter.Letter} was evicted for {reason}.");
    }
};

static async ValueTask IterateAlphabetAsync(
    Func<char, Task> asyncFunc)
{
    for (char letter = 'A'; letter <= 'Z'; ++letter)
    {
        await asyncFunc(letter);
    }

    Console.WriteLine();
}

var addLettersToCacheTask = IterateAlphabetAsync(letter =>
{
    MemoryCacheEntryOptions options = new()
    {
        AbsoluteExpirationRelativeToNow =
            TimeSpan.FromMilliseconds(MillisecondsAbsoluteExpiration)
    };

    _ = options.RegisterPostEvictionCallback(OnPostEviction);

    AlphabetLetter alphabetLetter =
        cache.Set(
            letter, new AlphabetLetter(letter), options);

    Console.WriteLine($"{alphabetLetter.Letter} was cached.");

    return Task.Delay(
        TimeSpan.FromMilliseconds(MillisecondsDelayAfterAdd));
});
await addLettersToCacheTask;

var readLettersFromCacheTask = IterateAlphabetAsync(letter =>
{
    if (cache.TryGetValue(letter, out object? value) &&
        value is AlphabetLetter alphabetLetter)
    {
        Console.WriteLine($"{letter} is still in cache. {alphabetLetter.Message}");
    }

    return Task.CompletedTask;
});
await readLettersFromCacheTask;

await host.RunAsync();

file record AlphabetLetter(char Letter)
{
    internal string Message =>
        $"The '{Letter}' character is the {Letter - 64} letter in the English alphabet.";
}
```

## HybridCache

Get a value with hybridCache:

```cs
public class SomeService(HybridCache cache)
{
    private HybridCache _cache = cache;

    public async Task<string> GetSomeInfoAsync(string name, int id, CancellationToken token = default)
    {
        return await _cache.GetOrCreateAsync(
            $"{name}-{id}", // Unique key to the cache entry
            async cancel => await GetDataFromTheSourceAsync(name, id, cancel),
            cancellationToken: token
        );
    }

    public async Task<string> GetDataFromTheSourceAsync(string name, int id, CancellationToken token)
    {
        string someInfo = $"someinfo-{name}-{id}";
        return someInfo;
    }
}
```

If you want to register it in your application, use the following code:

```cs
services.AddHybridCache(options =>
{
    options.DefaultEntryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromMinutes(10),
        LocalCacheExpiration = TimeSpan.FromMinutes(10)
    };
});

// And if you want to add redis, you can do it like that.
services
    .AddStackExchangeRedisCache(options =>
    {
        options.Configuration = configuration.GetConnectionString("Redis")!;
    });
```

## Cache Aside

You manually look for data if you don't find it in your cache.

```cs
public class CacheAsideProductService(IMemoryCache cache, IProductRepository repository)
    : IProductService
{
    public async Task<Product?> GetByIdAsync(int id)
    {
        // Define the cache key
        var cacheKey = $"product:{id}";

        // 1. Try to get the product from the cache
        var product = cache.Get<Product>(cacheKey);
        if (product is not null)
        {
            return product; // Cache hit
        }

        // 2. If not found, load the product from the database
        product = await repository.GetByIdAsync(id);
        if (product != null)
        {
            // 3. Update the cache with the fetched product
            cache.Set(cacheKey, product, new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
        }

        return product;
    }
}
```

## Read Through cache

Whenever you do a search, you look for the data in the cache which handles itelf. You don't do anything in the cache directly, you let it fill itself. The cache should handle getting new values.

Note: can be combined with write around and other types of cache.

```cs
public class ReadThroughCacheProductService(HybridCache cache, IProductRepository repository)
    : IProductService
{
    public async Task<Product?> GetByIdAsync(int id)
    {
        // Define the cache key
        var cacheKey = $"product:{id}";

        // Read-through cache implementation
        var product = await cache.GetOrCreateAsync<Product?>(
            cacheKey,
            // Data loader function to fetch from DB
            async token => await repository.GetByIdAsync(id),
            new HybridCacheEntryOptions
            {
                Expiration = TimeSpan.FromMinutes(10)
            });

        return product;
    }
}
```

### Write Around

When you write something, cache does not update itself. It updates only on read. Useful for cases with a lot of writes (example: logs or sensor data).

Problem is potential stale data if you don't check data often.

```cs
public class WriteAroundCacheProductService(IMemoryCache cache, IProductRepository repository)
    : IProductService
{
    public async Task<Product?> GetByIdAsync(int id)
    {
        // Define the cache key
        var cacheKey = $"product:{id}";

        // 1. Try to get the product from the cache
        var product = cache.Get<Product>(cacheKey);
        if (product is not null)
        {
            return product; // Cache hit
        }

        // 2. If not found, load the product from the database
        product = await repository.GetByIdAsync(id);
        if (product != null)
        {
            // 3. Update the cache with the fetched product
            cache.Set(cacheKey, product, new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
        }

        return product;
    }

    public async Task AddAsync(Product product)
    {
        // Add the product to the database. Cache is not updated
        await repository.AddAsync(product);
    }
}
```

## Write back

Data is written in cache and then there is a write in database. This approach can improve writes (they are done at a better time). Problem is that you need good conflict handling if you have written to multiple caches. Useful for shopping carts.

```cs
public class WriteBackCacheProductCartService(
    HybridCache cache,
    IProductCartRepository repository,
    IProductRepository productRepository,
    Channel<ProductCartDispatchEvent> channel)
{
    public async Task<ProductCartResponse?> GetByIdAsync(Guid id)
    {
        // Define the cache key
        var cacheKey = $"productCart:{id}";

        // Retrieve the ProductCart from the cache (or from the database if missing)
        var productCartResponse = await cache.GetOrCreateAsync<ProductCartResponse?>(
            cacheKey,
            // Data loader function to fetch from DB
            async token => 
            {
                var productCart = await repository.GetByIdAsync(id);
                return productCart?.MapToProductCartResponse();
            },
            new HybridCacheEntryOptions
            {
                Expiration = TimeSpan.FromMinutes(10)
            });

        return productCartResponse;
    }

    public async Task<ProductCartResponse> AddAsync(ProductCartRequest request)
    {
        var productCart = new ProductCart
        {
            Id = Guid.NewGuid(),
            UserId = request.UserId,
            CartItems = request.ProductCartItems.Select(x => x.MapToCartItem()).ToList()
        };
        
        var cacheKey = $"productCart:{productCart.Id}";

        // Add or update the ProductCart in the cache
        var productCartResponse = productCart.MapToProductCartResponse();
        await cache.SetAsync(cacheKey, productCartResponse);

        // Queue the ProductCart for database syncing
        await channel.Writer.WriteAsync(new ProductCartDispatchEvent(productCart, false));

        return productCartResponse;
    }
}

public record ProductCartDispatchEvent(ProductCart ProductCart, bool IsDeleted);

public class WriteBackCacheBackgroundService(IServiceScopeFactory scopeFactory,
    Channel<ProductCartDispatchEvent> channel) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var command in channel.Reader.ReadAllAsync(stoppingToken))
        {
            using var scope = scopeFactory.CreateScope();
            var repository = scope.ServiceProvider.GetRequiredService<IProductCartRepository>();
            
            if (command.IsDeleted)
            {
                await repository.DeleteAsync(command.ProductCart.Id);
                return;
            }

            var existingCart = await repository.GetByIdAsync(command.ProductCart.Id);
            if (existingCart is null)
            {
                await repository.AddAsync(command.ProductCart);
            }
            else
            {
                existingCart.CartItems = command.ProductCart.CartItems;
                await repository.UpdateAsync(existingCart);
            }
        }
    }
}
```

## Write through

Update both the cache and the database simultanuously.

```cs
public class WriteThroughCacheProductService(HybridCache cache, IProductRepository repository) : IProductService
{
    public async Task<Product?> GetByIdAsync(int id)
    {
        // Define the cache key
        var cacheKey = $"product:{id}";

        // Cache should always have the value but just in case we can check the database on a cache miss
        var product = await cache.GetOrCreateAsync<Product?>(
            cacheKey,
            // Data loader function to fetch from DB
            async token => await repository.GetByIdAsync(id),
            new HybridCacheEntryOptions
            {
                Expiration = TimeSpan.FromMinutes(10)
            });

        return product;
    }

    public async Task AddAsync(Product product)
    {
        // Add the product to the database
        await repository.AddAsync(product);

        // Write the product to the cache (write through)
        var cacheKey = $"product:{product.Id}";
        
        await cache.SetAsync(cacheKey, product, new HybridCacheEntryOptions
        {
            Expiration = TimeSpan.FromMinutes(10)
        });
    }
}
```

## Database result caching

Introduce caching in your queries in order to get results faster.

```cs
public async Task<Book?> GetBookAsync(Guid id, CancellationToken cancellationToken)
{
    var cacheKey = $"Book_{id}";

    if (_cache.TryGetValue(cacheKey, out Book? book))
    {
        return book;
    }
    
    book = await _context.Books
        .Include(b => b.Author)
        .FirstOrDefaultAsync(b => b.Id == id, cancellationToken);
        
    return book;
}
```

## Output Cache

### Basic version

The previous paragraph is about memory cache. You can also cache your outputs so that if an answer was already given, you can send it back. For that, you will need the package **Microsoft.AspNetCore.OutputCaching**.

```cs
var builder = WebApplication.CreateBuilder(args);

// Register OutputCache in DI
builder.Services.AddOutputCache();

var app = builder.Build();

// Add OutputCache Middleware
app.UseOutputCache(); // This is the important line.

app.MapControllers();

app.Run();
```

Classic API code.

```cs
[ApiController]
[Route("api/orders")]
public class OrdersController(OrdersDbContext db) : ControllerBase
{
    [HttpGet]
    [OutputCache(Duration = 120)] // You can then use Output cache on your endpoints. You can give parameters.
    public async Task<IActionResult> GetOrders(CancellationToken ct)
    {
        var orders = await db.Orders.ToListAsync(ct);
        return Ok(orders);
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetOrder(Guid id, CancellationToken ct)
    {
        var order = await db.Orders.FindAsync([id], ct);
        if (order is null) return NotFound();
        return Ok(order);
    }
}
```

Minimal API code:

```cs
app.MapGet("/api/orders", async (OrdersDbContext db, CancellationToken ct) =>
{
    var orders = await db.Orders.ToListAsync(ct);
    return Results.Ok(orders);
}).CacheOutput(x => x.Expire(TimeSpan.FromMinutes(2))); // You can give parameters here.
```

If you want to avoid having local parameters and want global policies / parameters. You can give names in your policies so that you can call them by name.

```cs
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("OrdersPolicy", policy =>
        policy
            .Expire(TimeSpan.FromMinutes(5))
            .SetVaryByHeader("Accept-Language")
            .Tag("orders")
    );

    options.AddPolicy("OrderDetailPolicy", policy =>
        policy
            .Expire(TimeSpan.FromMinutes(2))
            .SetVaryByRouteValue("id")
            .Tag("orders")
    );
});
```

```cs
[OutputCache(PolicyName = "OrdersPolicy")]
```

or for minimal apis:

```cs
.CacheOutput("OrderDetailPolicy")
```

### Additional parameters

You may want to have different values cached depending on which endpoint was called or which header you have.

```cs
options.AddPolicy("PagedOrdersPolicy", policy =>
    policy
        .Expire(TimeSpan.FromMinutes(5))
        .SetVaryByQuery("page", "pageSize", "status")
        .Tag("orders")
);
```

### Cache eviction

We have seen that we can parametrize cache eviction and have different values by tag. It is also possible to set cache eviction by tag:

```cs
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("OrdersPolicy", policy =>
        policy
            .Expire(TimeSpan.FromMinutes(5))
            .Tag("orders")
    );

    options.AddPolicy("OrderDetailPolicy", policy =>
        policy
            .Expire(TimeSpan.FromMinutes(2))
            .SetVaryByRouteValue("id")
            .Tag("orders")
            .Tag("order-detail")
    );
});
```

Cache eviction allows us to evict order value when the order is passed for example. Here we evict the values by tag.

```cs
public async Task<IActionResult> CreateOrder(
    [FromBody] CreateOrderRequest request,
    CancellationToken ct)
{
    var order = new Order(Guid.NewGuid(), request.CustomerName, request.TotalAmount, "Pending");
    db.Orders.Add(order);
    await db.SaveChangesAsync(ct);

    // Evict all cache entries tagged with "orders"
    await cache.EvictByTagAsync("orders", ct);

    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}

// Do the same for PUT and DELETE
```

Here, we evict values by Id.

```cs
[HttpPut("{id:guid}")]
public async Task<IActionResult> UpdateOrder(
    Guid id,
    [FromBody] UpdateOrderRequest request,
    CancellationToken ct)
{
    var order = await db.Orders.FindAsync([id], ct);
    if (order is null) return NotFound();

    db.Orders.Entry(order).CurrentValues.SetValues(request);
    await db.SaveChangesAsync(ct);

    // Evict only the cache entry for this specific order
    await cache.EvictByTagAsync($"order-{id}", ct);

    return NoContent();
}
```

### In memory vs in database

By default, output cache is in memory. It is not shared between users. You can add a redis database for output cache.

```cs
var redisConnectionString = builder.Configuration.GetConnectionString("Redis")!;

builder.Services.AddStackExchangeRedisOutputCache(options =>
{
    options.Configuration = redisConnectionString;
    options.InstanceName = "OrdersApi:";
});

builder.Services.AddOutputCache(options =>
{
    // ...
});
```

### Risks

Caching authenticated API responses is ricky. User B might get the answer of user A which is a security risk. ==> Do not cache authenticated requests by default.

A possible solution would be to use VaryByValue and give a unique value for each user (for example, add userId):

```cs
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("UserOrdersPolicy", policy =>
        policy
            .Expire(TimeSpan.FromMinutes(2))
            .Tag("orders")
            .SetVaryByValue(ctx =>
            {
                // Extract the user ID from the JWT claims
                var userId = ctx.HttpContext.User.FindFirstValue(ClaimTypes.NameIdentifier)
                             ?? "anonymous";

                return new KeyValuePair<string, string>("userId", userId);
            })
    );
});
```

```cs
[Authorize]
[HttpGet("my")]
[OutputCache(PolicyName = "UserOrdersPolicy")]
public async Task<IActionResult> GetMyOrders(CancellationToken ct)
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier)!;

    var orders = await db.Orders
        .Where(o => o.UserId == userId)
        .ToListAsync(ct);

    return Ok(orders);
}
```
