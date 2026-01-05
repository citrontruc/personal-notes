# Cache

## Table of Content

- [Cache](#cache)
  - [Table of Content](#table-of-content)
  - [Example](#example)
  - [HybridCache](#hybridcache)
  - [Cache Aside](#cache-aside)
  - [Read Through cache](#read-through-cache)
    - [Write Around](#write-around)
  - [Write back](#write-back)
  - [Write through](#write-through)
  - [Database result caching](#database-result-caching)

## Example

You have to configure cache in your application with timeout and parameters. You can then put data in cache. When you want to access them, just cache.TryGetValue.

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

Whenever you do a search, you look for the data in the cache which handles itelf. You don't do anything in the cache directly, you let it fill itself.

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

Data is written in cache and then there is a write in database. This approach can improve writes (they are done at a better time). Problem is that you need good conflict handling. Useful for shopping carts.

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
