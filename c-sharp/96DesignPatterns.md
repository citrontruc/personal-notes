# Design Patterns

## Table of content

- [Design Patterns](#design-patterns)
  - [Table of content](#table-of-content)
  - [Decorator](#decorator)
  - [Option design pattern](#option-design-pattern)
    - [Option pattern and validation](#option-pattern-and-validation)
  - [Extension](#extension)
    - [Classic approach](#classic-approach)
    - [Modern approach](#modern-approach)
    - [Example with properties](#example-with-properties)
    - [static methods](#static-methods)
    - [Real world example to extract fields of requests](#real-world-example-to-extract-fields-of-requests)
    - [Good practice](#good-practice)
  - [Dependency Injection](#dependency-injection)
    - [🛠️ Service Lifetimes](#️-service-lifetimes)
  - [Dependency Injection are your friend](#dependency-injection-are-your-friend)
    - [What about APIs?](#what-about-apis)
  - [Service Locator](#service-locator)
  - [Singleton](#singleton)
  - [Repository pattern](#repository-pattern)
  - [Declaration pattern](#declaration-pattern)
  - [Feature flag](#feature-flag)
  - [Result Pattern](#result-pattern)

## Decorator

There are no specific keywords. You just have to implement it.

```cs
// Concrete Decorator 1
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription()
    {
        // Add new description to the existing one
        return base.GetDescription() + ", Milk";
    }

    public override double GetCost()
    {
        // Add new cost to the existing one
        return base.GetCost() + 1.50;
    }
}
```

If you want to insert an error client in a messaging service:

```cs
public class ExceptionReportingMessenger : MessengerDecorator
{
    private readonly IExceptionReportingService _reportingService;

    public ExceptionReportingMessenger(
        IMessenger underlying,
        IExceptionReportingService reportingService ) :
        base( underlying )
    {
        this._reportingService = reportingService;
    }

    public override void Send( Message message )
    {
        try
        {
            this.Underlying.Send( message );
        }
        catch ( Exception e )
        {
            this._reportingService.ReportException( "Failed to send message", e );

            throw;
        }
    }

    public override Message Receive()
    {
        try
        {
            return this.Underlying.Receive();
        }
        catch ( Exception e )
        {
            this._reportingService.ReportException( "Failed to receive message", e );

            throw;
        }
    }
}
```

## Option design pattern

You can find a tutorial over here: <https://dev.to/vimaltwit/understanding-the-options-pattern-in-net-core-j1j>

Imagine, you have authentication information stored in appsettings.json:

```json
{
  "MyOptions": {
    "Endpoint": "https://api.example.com",
    "TimeoutSeconds": 30,
    "EnableCaching": true
  }
}
```

You can create an option class that is a dataclass to get your options.

```cs
using System.ComponentModel.DataAnnotations;

public sealed class MyOptions
{
    public const string SectionName = "MyOptions";

    [Required, Url]
    public string Endpoint { get; init; } = default!;

    [Range(1, 300)]
    public int TimeoutSeconds { get; init; }

    public bool EnableCaching { get; init; }
}

// You can now register your option:
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddOptions<MyOptions>()
    .Bind(builder.Configuration.GetSection(MyOptions.SectionName))
    .ValidateDataAnnotations()
    .Validate(o => o.TimeoutSeconds > 0, "Timeout must be > 0")
    .ValidateOnStart();

var app = builder.Build();

// You can then use your options in your service:
public sealed class MyService
{
    private MyOptions _options;

    public MyService(IOptionsMonitor<MyOptions> monitor)
    {
        _options = monitor.CurrentValue;

        monitor.OnChange((opts, _) =>
        {
            _options = opts;
        });
    }
}
```

**IOptionsMonitor** is the reloadable options object. It change automatically when values change.

### Option pattern and validation

In your options, you want to fail fast in order to see problems as early as possible. You could add validation to your data model, but that could be a problem because we want Separation of Concern.

There is a library called FluentValidation <https://docs.fluentvalidation.net/en/latest/> that lets you write validation rules:

```cs
public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(x => x.Surname).NotEmpty();
        RuleFor(x => x.Forename).NotEmpty().WithMessage("Please specify a first name");
        RuleFor(x => x.Discount).NotEqual(0).When(x => x.HasDiscount);
        RuleFor(x => x.Address).Length(20, 250);
        RuleFor(x => x.Postcode).Must(BeAValidPostcode).WithMessage("Please specify a valid postcode");
    }

    // On passe une règle de validation custom ici.
    private bool BeAValidPostcode(string postcode)
    {
        // custom postcode validating logic goes here
    }
}
```

You can find under a custom validator for Github parameters:

```cs
public class GitHubSettings
{
    public const string ConfigurationSection = "GitHubSettings";

    public string BaseUrl { get;init; }
    public string AccessToken { get; init; }
    public string RepositoryName { get; init; }
}

public class GitHubSettingsValidator : AbstractValidator<GitHubSettings>
{
    public GitHubSettingsValidator()
    {
        RuleFor(x => x.BaseUrl).NotEmpty();

        RuleFor(x => x.BaseUrl)
            .Must(baseUrl => Uri.TryCreate(BaseUrl, UriKind.Absolute, out _))
            .When(x => !string.IsNullOrWhiteSpace(x.baseUrl))
            .WithMessage($"{nameof(GitHubSettings.BaseUrl)} must be a valid URL");

        RuleFor(x => x.AccessToken)
            .NotEmpty();

        RuleFor(x => x.RepositoryName)
            .NotEmpty();
    }
}
```

We now need to integrate our Fluent Validation in our system:

```cs
using FluentValidation;
using Microsoft.Extensions.Options;

public class FluentValidateOptions<TOptions>
    : IValidateOptions<TOptions>
    where TOptions : class
{
    private readonly IServiceProvider _serviceProvider;
    private readonly string? _name;

    public FluentValidateOptions(IServiceProvider serviceProvider, string? name)
    {
        _serviceProvider = serviceProvider;
        _name = name;
    }

    public ValidateOptionsResult Validate(string? name, TOptions options)
    {
        if (_name is not null && _name != name)
        {
            return ValidateOptionsResult.Skip;
        }

        ArgumentNullException.ThrowIfNull(options);

        using var scope = _serviceProvider.CreateScope();

        var validator = scope.ServiceProvider.GetRequiredService<IValidator<TOptions>>();

        var result = validator.Validate(options);
        if (result.IsValid)
        {
            return ValidateOptionsResult.Success;
        }

        var type = options.GetType().Name;
        var errors = new List<string>();

        foreach (var failure in result.Errors)
        {
            errors.Add($"Validation failed for {type}.{failure.PropertyName} " +
                       $"with the error: {failure.ErrorMessage}");
        }

        return ValidateOptionsResult.Fail(errors);
    }
}
```

To make our Validation easier, we can add a few extensions:

```cs
// Method to add to our builder the option validation.
public static class OptionsBuilderExtensions
{
    public static OptionsBuilder<TOptions> ValidateFluentValidation<TOptions>(
        this OptionsBuilder<TOptions> builder)
        where TOptions : class
    {
        builder.Services.AddSingleton<IValidateOptions<TOptions>>(
            serviceProvider => new FluentValidateOptions<TOptions>(
                serviceProvider,
                builder.Name));

        return builder;
    }
}

// Method to add fluent validation.
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddOptionsWithFluentValidation<TOptions>(
        this IServiceCollection services,
        string configurationSection)
        where TOptions : class
    {
        services.AddOptions<TOptions>()
            .BindConfiguration(configurationSection)
            .ValidateFluentValidation() // Configure FluentValidation validation
            .ValidateOnStart(); // Validate options on application start

        return services;
    }
}
```

You then need to register your option pattern:

```cs
// Register the validator
builder.Services.AddScoped<IValidator<GitHubSettings>, GitHubSettingsValidator>();

// Configure options with validation
builder.Services.AddOptions<GitHubSettings>()
    .BindConfiguration(GitHubSettings.ConfigurationSection)
    .ValidateFluentValidation() // Configure FluentValidation validation
    .ValidateOnStart();

// Use the convenience extension (other option)
// builder.Services.AddOptionsWithFluentValidation<GitHubSettings>(GitHubSettings.ConfigurationSection);
```

## Extension

### Classic approach

You can add methods to a class later via extensions. You have to make them static.

```cs
// 1. Must be a static class
public static class StringExtensions 
{
    // 2. Must be a static method
    // 3. The first parameter uses the 'this' keyword followed by the type to extend (string)
    public static string Reverse(this string s)
    {
        char[] charArray = s.ToCharArray();
        Array.Reverse(charArray);
        return new string(charArray);
    }
}

// Your strings now have an added method.
string value = "Test";
value.Reverse();
```

### Modern approach

There is a more modern approach to group multiple extensions together. Works with C# 14 and more recent versions.

```cs
// Old version of extension. Uses the this keyword.
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string value)
    {
        return string.IsNullOrEmpty(value);
    }
    
    public static string Truncate(this string value, int maxLength)
    {
        if (string.IsNullOrEmpty(value) || value.Length <= maxLength)
        {
            return value;
        }
            
        return value.Substring(0, maxLength);
    }
}

// Modern approach is to use the extension keyword in order to separate the extension methods from the element you are extending:
public static class StringExtensions
{
    extension(string value)
    {
        public bool IsNullOrEmpty()
        {
            return string.IsNullOrEmpty(value);
        }
        
        public string Truncate(int maxLength)
        {
            if (string.IsNullOrEmpty(value) || value.Length <= maxLength)
            {
                return value;
            }
                
            return value.Substring(0, maxLength);
        }
    }
}
```

### Example with properties

With the modern approach for extensions, you can add properties to elements easily:

```cs
public static class CollectionExtensions
{
    extension<T>(IEnumerable<T> source)
    {
        public bool IsEmpty => !source.Any();

        public bool HasItems => source.Any();

        public int Count => source.Count();

        public IEnumerable<T> WhereNotNull() =>
            source.Where(item => item != null);
        
        public Dictionary<TKey, List<T>> GroupToDictionary<TKey>(
            Func<T, TKey> keySelector) where TKey : notnull =>
            source.GroupBy(keySelector)
                  .ToDictionary(g => g.Key, g => g.ToList());
    }
}

public void ProcessOrders(IEnumerable<Order> orders)
{
    if (orders.IsEmpty)
    {
        Console.WriteLine("No orders to process");
        return;
    }

    foreach (var order in orders)
    {
        // Process order
    }
}

// We can then do that:
var products = _productService.GetAll();

var productsByCategory = products
    .WhereNotNull()
    .Where(p => p.IsAvailable)
    .GroupToDictionary(p => p.Category);

foreach (var category in productsByCategory)
{
    Console.WriteLine($"{category.Key}: {category.Value.Count} products");
}
```

### static methods

With extensions, it is also possible to add static methods to the class and not the instance of the class.

```cs
public static class ProductExtensions
{
    extension(Product)
    {
        public static Product CreateDefault() =>
            new Product
            {
                Name = "Unnamed Product",
                Price = 0,
                StockQuantity = 0,
                Category = "Uncategorized",
                CreatedDate = DateTime.UtcNow
            };

        public static bool IsValidPrice(decimal price) =>
            price >= 0 && price <= 1000000;

        public static string DefaultCategory => "General";
    }
}
```

### Real world example to extract fields of requests

```cs
public static class ApiHttpContextExtensions
{
    extension(HttpContext context)
    {
        public string CorrelationId =>
            context.Request.Headers["X-Correlation-ID"].FirstOrDefault()
                ?? Guid.NewGuid().ToString();

        public string ClientIp =>
            context.Request.Headers["X-Forwarded-For"].FirstOrDefault()
                ?? context.Connection.RemoteIpAddress?.ToString()
                ?? "Unknown";

        public bool IsApiRequest =>
            context.Request.Path.StartsWithSegments("/api");

        public string? GetBearerToken()
        {
            var authHeader = context.Request.Headers["Authorization"].FirstOrDefault();
            if (authHeader?.StartsWith("Bearer ") == true)
            {
                return authHeader.Substring("Bearer ".Length).Trim();
            }
            return null;
        }

        public T? GetQueryParameter<T>(string key)
        {
            if (context.Request.Query.TryGetValue(key, out var value))
            {
                try
                {
                    return (T?)Convert.ChangeType(value.ToString(), typeof(T));
                }
                catch
                {
                    return default;
                }
            }
            return default;
        }

        public void AddResponseHeader(string key, string value)
        {
            context.Response.Headers[key] = value;
        }
    }

    extension(HttpContext)
    {
        public static bool IsValidPath(string path) =>
            !string.IsNullOrWhiteSpace(path) && path.StartsWith("/");
    }
}
```

### Good practice

Regroup extensions on similar types and interfaces. Put them in the Utils folder maybe?

```cs
public static class CollectionExtensions
{
    extension<T>(IEnumerable<T> source)
    {
        public bool IsEmpty => !source.Any();

        public bool HasItems => source.Any();

        public int Count => source.Count();
    }

    extension(IEnumerable<string> source)
    {
        public string JoinWithComma() => string.Join(", ", source);
        public IEnumerable<string> NonEmpty() => source.Where(s => !string.IsNullOrEmpty(s));
    }

    extension<T>(List<T> list)
    {
        public void AddIfNotExists(T item)
        {
            if (!list.Contains(item))
            {
                list.Add(item);
            }
        }
    }
}

// Don't forget to document your extensions:
extension(Product product)
{
    /// <summary>
    /// Gets whether the product is currently available for purchase.
    /// </summary>
    /// <remarks>
    /// A product is available if its stock quantity is greater than zero.
    /// </remarks>
    public bool IsAvailable => product.StockQuantity > 0;
}
```

## Dependency Injection

You can create a webapp objects and add all the services to this webapp. When you do that, the webapp will handle itself dependency resolutions.

### 🛠️ Service Lifetimes

---

**TL;DR**: Use Transient if you need to create multiple instances of an instance (example: in different threads). Use scoped for database instances to avoid unsynchronized instances. Use singletons when you have thread safe pure logic. No dependencies whatsoever.

---

The three methods register services with different lifetimes in the DI container. The choice depends on how you want the service's state to be managed and shared throughout your application.

```cs
services.AddTransient<TService, TImplementation>()
```

**Definition**: A new instance of the service is created every time it is requested, regardless of where the request is coming from.

When to use it:

For lightweight, stateless services that perform a single operation.

For services that should not share state and need to be isolated from other consumers.

Examples: Simple utility classes, or a service that calculates a value and immediately returns it.

```cs
services.AddScoped<TService, TImplementation>()
```

**Definition**: A single instance of the service is created per scope. In a typical ASP.NET Core web application, the scope is usually the client's request (HTTP request).

This means that within one HTTP request, the service will be the same instance.

If the same client sends another request, a brand new instance will be created for that second request.

When to use it:

For services that need to maintain state during a single request but should not persist across requests.

This is the most common lifetime for services interacting with request-specific data, such as a database context (like DbContext in Entity Framework Core).

Examples: A service that coordinates multiple repository operations for a single transaction, or a service that holds request-specific user information.

```cs
services.AddSingleton<TService, TImplementation>()
```

**Definition**: A single instance of the service is created the first time it is requested (or when Startup.ConfigureServices runs, depending on the registration method) and that same instance is reused for every subsequent request for the entire lifetime of the application.

When to use it:

For services that are stateless or must have a globally shared state (like configuration settings or a cache).

For services that are expensive to create.

**⚠️ Important Note**: Singleton services must never take a dependency on a Scoped or Transient service if that dependency holds request-specific state, as this can lead to subtle bugs where the Singleton service holds onto an outdated or incorrect Scoped instance across requests.

Examples: Logging services, configuration managers, or an in-memory caching service.

```cs
var builder = WebApplication.CreateBuilder(args);

// Add documentation generation
builder.Services.AddOpenApi();
// Add services to be mapped.
builder.Services.AddSingleton<IMusicRepository, InMemoryMusicRepository>();
builder.Services.AddSingleton<AlbumParameters, AlbumParameters>();
```

## Dependency Injection are your friend

There is a microsoft library to which you can register your services. You can then ask a ServiceCollection to resolve services for you.

```cs
public interface IMessageSender
{
    void Send(string message);
}

public class EmailSender : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class NotificationService
{
    private readonly IMessageSender _sender;

    // Dependency is injected
    public NotificationService(IMessageSender sender)
    {
        _sender = sender;
    }

    public void Notify(string msg) => _sender.Send(msg);
}
```

You can then create your component which will create all the instances. Note : the ServiceCollection can handle multiple constructors.

```cs
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();

// If you want to change to SmsSender, you can do it here.
services.AddTransient<IMessageSender, EmailSender>();
services.AddTransient<NotificationService>();

var provider = services.BuildServiceProvider();

var svc = provider.GetRequiredService<NotificationService>();
svc.Notify("Hello");
```

Careful! You can have conflicts if you register twice a service with the same interface (exemple: if you register two IMessageSender, the lib won't know which one to use). You can add functions for that but it is not practical.

**Service lifetime**: Transient, Scoped or Singleton.

- Transient services are created each time they're requested from the service container. They are disposed at the end of requests. Services are resolved and constructed every time.
- Scoped are created once per client request. Disposed at end of request. Be careful with scoped services.
- Singleton Created first time they are requested most times. We use the same instance for every subsequent request. Must be thread safe. Careful with memory use.

### What about APIs?

For apis, instead of declaring services, you can just declare controllers as controllers and let service discovery naturally take place.

Example underneath for an ApiController. This version is an older version and is gradually being moved out.

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Options;
using MusicAlbumApi.Data;
using MusicAlbumApi.Models;
using MusicAlbumApi.Options;

namespace MusicAlbumApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AlbumsController : ControllerBase
{
    private readonly MusicDbContext _context;
    private readonly PaginationOptions _paginationOptions;
    private readonly ApiSettingsOptions _apiSettings;

    public AlbumsController(
        MusicDbContext context,
        IOptions<PaginationOptions> paginationOptions,
        IOptions<ApiSettingsOptions> apiSettings)
    {
        _context = context;
        _paginationOptions = paginationOptions.Value;
        _apiSettings = apiSettings.Value;
    }

    // GET: api/albums
    [HttpGet]
    public async Task<ActionResult<object>> GetAlbums(
        [FromQuery] string? artist = null,
        [FromQuery] string? genre = null,
        [FromQuery] int? page = null,
        [FromQuery] int? pageSize = null)
    {
        // Use global settings with validation
        var currentPage = page ?? 1;
        var currentPageSize = pageSize ?? _paginationOptions.DefaultPageSize;
        
        // Enforce max and min page size
        if (currentPageSize > _paginationOptions.MaxPageSize)
            currentPageSize = _paginationOptions.MaxPageSize;
        
        if (currentPageSize < _paginationOptions.MinPageSize)
            currentPageSize = _paginationOptions.MinPageSize;

        var query = _context.Albums.AsQueryable();

        if (!string.IsNullOrWhiteSpace(artist))
            query = query.Where(a => a.Artist.Contains(artist));

        if (!string.IsNullOrWhiteSpace(genre))
            query = query.Where(a => a.Genre != null && a.Genre.Contains(genre));

        var totalCount = await query.CountAsync();
        var totalPages = (int)Math.Ceiling(totalCount / (double)currentPageSize);

        var albums = await query
            .OrderByDescending(a => a.CreatedAt)
            .Skip((currentPage - 1) * currentPageSize)
            .Take(currentPageSize)
            .ToListAsync();

        // Return paginated response with metadata
        return Ok(new
        {
            data = albums,
            pagination = new
            {
                currentPage,
                pageSize = currentPageSize,
                totalCount,
                totalPages,
                hasNextPage = currentPage < totalPages,
                hasPreviousPage = currentPage > 1
            }
        });
    }

    // GET: api/albums/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Album>> GetAlbum(int id)
    {
        var album = await _context.Albums.FindAsync(id);

        if (album == null)
            return NotFound(new { message = $"Album with ID {id} not found" });

        return Ok(album);
    }

    // POST: api/albums
    [HttpPost]
    public async Task<ActionResult<Album>> CreateAlbum(Album album)
    {
        album.CreatedAt = DateTime.UtcNow;
        album.UpdatedAt = DateTime.UtcNow;

        _context.Albums.Add(album);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetAlbum), new { id = album.Id }, album);
    }

    // PUT: api/albums/5
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateAlbum(int id, Album album)
    {
        if (id != album.Id)
            return BadRequest(new { message = "ID mismatch" });

        var existingAlbum = await _context.Albums.FindAsync(id);
        if (existingAlbum == null)
            return NotFound(new { message = $"Album with ID {id} not found" });

        existingAlbum.Title = album.Title;
        existingAlbum.Artist = album.Artist;
        existingAlbum.Genre = album.Genre;
        existingAlbum.ReleaseDate = album.ReleaseDate;
        existingAlbum.Rating = album.Rating;
        existingAlbum.TrackCount = album.TrackCount;
        existingAlbum.CoverImageUrl = album.CoverImageUrl;
        existingAlbum.Description = album.Description;
        existingAlbum.UpdatedAt = DateTime.UtcNow;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!await AlbumExists(id))
                return NotFound();
            throw;
        }

        return NoContent();
    }

    // DELETE: api/albums/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteAlbum(int id)
    {
        var album = await _context.Albums.FindAsync(id);
        if (album == null)
            return NotFound(new { message = $"Album with ID {id} not found" });

        _context.Albums.Remove(album);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    // GET: api/albums/config
    [HttpGet("config")]
    public ActionResult<object> GetConfiguration()
    {
        return Ok(new
        {
            pagination = _paginationOptions,
            apiSettings = _apiSettings
        });
    }

    private async Task<bool> AlbumExists(int id)
    {
        return await _context.Albums.AnyAsync(e => e.Id == id);
    }
}
```

In our case above, we a a Program.cs equal to this

```cs
using Microsoft.EntityFrameworkCore;
using MusicAlbumApi.Data;
using MusicAlbumApi.Options;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Configure Options pattern for global settings
builder.Services.Configure<PaginationOptions>(
    builder.Configuration.GetSection(PaginationOptions.Section));

builder.Services.Configure<ApiSettingsOptions>(
    builder.Configuration.GetSection(ApiSettingsOptions.Section));

// Add DbContext with SQLite (you can change to SQL Server or other providers)
builder.Services.AddDbContext<MusicDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

See project for recommended way to do things with Apis. Now in minimal APIs, everything is an endpoint and can be used as an endpoint so we still have a **AddEndpointsApiExplorer** but we use **MapGet** and **PostGet**.

## Service Locator

Generally discouraged. We much rather have dependency injection.

```cs
/*
Service locator who returns which services exist and are registered.
*/

public static class ServiceLocator
{
    private static Dictionary<Type, object> _services = new Dictionary<Type, object>();

    public static void Register<T>(T service)
    {
        if (_services.ContainsKey(typeof(T)))
            throw new InvalidOperationException(
                $"Service of type {typeof(T)} is already registered."
            );
        if (service is null)
        {
            throw new NullReferenceException($"Service of type {typeof(T)} is null.");
        }
        _services[typeof(T)] = service;
    }

    public static T Get<T>()
    {
        if (!_services.ContainsKey(typeof(T)))
            throw new InvalidOperationException($"Service of type {typeof(T)} is not registered.");
        return (T)_services[typeof(T)];
    }

    public static void Unregister<T>()
    {
        if (_services.ContainsKey(typeof(T)))
            _services.Remove(typeof(T));
    }

    public static void Reset()
    {
        _services.Clear();
    }
}
```

## Singleton

```cs
/*
A class to create singletons and make sure that any duplicate instance gets deleted.
There must be only one singleton per scene.
*/

using UnityEngine;

public class Singleton<T> : MonoBehaviour
    where T : Component
{
    #region Singleton access
    private static T _instance;
    public static T Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = (T)FindFirstObjectByType(typeof(T));
                if (_instance == null)
                {
                    SetupInstance();
                }
            }
            return _instance;
        }
    }
    #endregion

    #region Monobehaviour methods
    public virtual void Awake()
    {
        RemoveDuplicates();
    }

    private static void SetupInstance()
    {
        _instance = (T)FindFirstObjectByType(typeof(T));
        if (_instance == null)
        {
            GameObject gameObj = new GameObject();
            gameObj.name = typeof(T).Name;
            _instance = gameObj.AddComponent<T>();
            //DontDestroyOnLoad(gameObj);
        }
    }
    #endregion

    private void RemoveDuplicates()
    {
        if (_instance == null)
        {
            _instance = this as T;
            //DontDestroyOnLoad(gameObject);
        }
        else if (_instance != this)
        {
            Destroy(gameObject);
        }
    }
}
```

We can have a simplified version of the Singleton with a lazy implementation:

```cs
public sealed class Singleton
{
    private static Lazy<Singleton> _instance = new Lazy<Singleton>(() => Singleton());
    public static Singleton Instance => _instance.Value;

    private Singleton() { }
}
```

## Repository pattern

Repository Pattern: An abstraction layer between your application logic and data access. It encapsulates the logic for retrieving and storing domain objects, providing a collection-like interface. The repository knows about your domain models and business logic.

**In API, the view is the JSON response.**

DbContext (in Entity Framework): The actual mechanism that communicates with the database. It handles connections, change tracking, and translating LINQ queries into SQL.

- Controller: Routes HTTP requests, handles status codes, NOT business logic
- Model: Just the data classes (Product, DTOs)
- Service: Business logic and validation
- Repository + DbContext: Data access (this is where database operations happen)

```cs
// Domain Model
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public bool IsActive { get; set; }
}

// Repository Interface (defines contract)
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetAllActiveAsync();
    Task<IEnumerable<Product>> SearchByNameAsync(string searchTerm);
    Task AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
}

// Repository Implementation
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<IEnumerable<Product>> GetAllActiveAsync()
    {
        return await _context.Products
            .Where(p => p.IsActive)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }

    public async Task<IEnumerable<Product>> SearchByNameAsync(string searchTerm)
    {
        return await _context.Products
            .Where(p => p.Name.Contains(searchTerm) && p.IsActive)
            .ToListAsync();
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var product = await GetByIdAsync(id);
        if (product != null)
        {
            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
        }
    }
}

// DbContext (handles database communication)
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) 
        : base(options) { }

    public DbSet<Product> Products { get; set; }
}

// Application Service (uses repository)
public class ProductService
{
    private readonly IProductRepository _repository;

    public ProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        // Business logic can go here
        return await _repository.GetByIdAsync(id);
    }

    public async Task<bool> CreateProductAsync(string name, decimal price)
    {
        // Validation and business rules
        if (string.IsNullOrWhiteSpace(name) || price <= 0)
            return false;

        var product = new Product
        {
            Name = name,
            Price = price,
            IsActive = true
        };

        await _repository.AddAsync(product);
        return true;
    }
}
```

## Declaration pattern

It allows you to both check the type of an object and declare a variable of that type in a single statement.

The pattern is particularly useful in scenarios where you need to handle different types of objects differently.

Example:

```cs
// Do not write:
if (product != null && product is Electronics){
    // Convert than apply transformation.
    var electronics = (Electronics)product;
    electronics.Price *= 0.8m;
}

// write this instead
// Check and transform.
if (product is Electronics electronics){
    electronics *=0.8m;
}
```

## Feature flag

Allows you to declare which features should be used and which one should be held back for later. You can add more or less complex conditions to your feature flags in order to choose when the features are enabled.

This is mostly possible with the appsettings.json file. It uses the FeatureManagement package.

```json
{
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    },

    // Define feature flags in a JSON file.
    "feature_management": {
        "feature_flags": [
            {
                "id": "FeatureT",
                "enabled": false
            },
            {
                "id": "FeatureU",
                "enabled": true,
                "conditions": {}
            },
            {
                "id": "FeatureV",
                "enabled": true,
                "conditions": {
                    "requirement_type": "All",
                    "client_filters": [
                        {  
                            "name": "Microsoft.TimeWindow",
                            "parameters": {
                                "Start": "Sun, 01 Jun 2025 13:59:59 GMT",
                                "End": "Fri, 01 Aug 2025 00:00:00 GMT"
                            }
                        },
                        {
                            "name": "Microsoft.Percentage",
                            "parameters": {
                                "Value": "50"
                            }
                        }
                    ]
                }
            }
        ]
    }
}
```

Once you have all your feature flags detailed in your code, you must register the FeatureManagement service (note: not shown here, you can add feature filters):

```cs
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
}
```

You can then evaluate your feature flags in your code:

```cs
public enum Features
{
    NewSearch,
    BetaExport
}

await _features.IsEnabledAsync(Features.NewSearch.ToString());
```

There are some nice writings for the MVC framework that you can use.

## Result Pattern

Don't throw errors for parts of the code who are not code-breaking. Having too many errors make it difficult to reason with the whole of your codebase. Keep errors only for exceptional cases.

We want our programs to fail gracefully and log their results before terminating. In order to do so, we should create a result class that catches errors and communicates what happened.

```cs
public sealed record Error(string Code, string Description)
{
    public static readonly Error None = new(string.Empty, string.Empty);
}

public class Result
{
    private Result(bool isSuccess, Error error)
    {
        if (isSuccess && error != Error.None ||
            !isSuccess && error == Error.None)
        {
            throw new ArgumentException("Invalid error", nameof(error));
        }

        IsSuccess = isSuccess;
        Error = error;
    }

    public bool IsSuccess { get; }

    public bool IsFailure => !IsSuccess;

    public Error Error { get; }

    public static Result Success() => new(true, Error.None);

    public static Result Failure(Error error) => new(false, error);
}
```

Another element to handle is the fail fast principal. When you have a configuration that does not work, throw an error when validating the class (at the very beginning) instead of when we try to use an object later down the line:

```cs
// Define our errors
public static class FollowerErrors
{
    public static readonly Error SameUser = new Error(
        "Followers.SameUser", "Can't follow yourself");

    public static readonly Error NonPublicProfile = new Error(
        "Followers.NonPublicProfile", "Can't follow non-public profiles");

    public static readonly Error AlreadyFollowing = new Error(
        "Followers.AlreadyFollowing", "Already following");
}

// Error with an argument.
public static class FollowerErrors
{
    public static Error NotFound(Guid id) => new Error(
        "Followers.NotFound", $"The follower with Id '{id}' was not found");
}

// Usse the Result pattern
public sealed class FollowerService
{
    private readonly IFollowerRepository _followerRepository;

    public FollowerService(IFollowerRepository followerRepository)
    {
        _followerRepository = followerRepository;
    }

    public async Task<Result> StartFollowingAsync(
        User user,
        User followed,
        DateTime utcNow,
        CancellationToken cancellationToken = default)
    {
        if (user.Id == followed.Id)
        {
            return Result.Failure(FollowerErrors.SameUser);
        }

        if (!followed.HasPublicProfile)
        {
            return Result.Failure(FollowerErrors.NonPublicProfile);
        }

        if (await _followerRepository.IsAlreadyFollowingAsync(
                user.Id,
                followed.Id,
                cancellationToken))
        {
            return Result.Failure(FollowerErrors.AlreadyFollowing);
        }

        var follower = Follower.Create(user.Id, followed.Id, utcNow);

        _followerRepository.Insert(follower);

        return Result.Success();
    }
}
```

In order to combine the result pattern with an API, you can use the following code:

```cs
app.MapPost(
    "users/{userId}/follow/{followedId}",
    (Guid userId, Guid followedId, FollowerService followerService) =>
    {
        var result = await followerService.StartFollowingAsync(
            userId,
            followedId,
            DateTime.UtcNow);

        if (result.IsFailure)
        {
            return Results.BadRequest(result.Error);
        }

        return Results.NoContent();
    });
```
