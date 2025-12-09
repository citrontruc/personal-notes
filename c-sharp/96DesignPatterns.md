# Design Patterns

## Table of content

- [Design Patterns](#design-patterns)
  - [Table of content](#table-of-content)
  - [Decorator](#decorator)
  - [Extension](#extension)
  - [Dependency Injection](#dependency-injection)
    - [🛠️ Service Lifetimes](#️-service-lifetimes)
  - [Dependency Injection are your friend](#dependency-injection-are-your-friend)
    - [What about APIs?](#what-about-apis)
  - [Service Locator](#service-locator)
  - [Singleton](#singleton)
  - [Repository pattern](#repository-pattern)
  - [Declaration pattern](#declaration-pattern)

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

## Extension

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

## Dependency Injection

You can create a webapp objects and add all the services to this webapp. When you do that, the webapp will handle itself dependency resolutions.

### 🛠️ Service Lifetimes

The three methods register services with different lifetimes in the DI container. The choice depends on how you want the service's state to be managed and shared throughout your application.

```cs
services.AddTransient<TService, TImplementation>()
```

Definition: A new instance of the service is created every time it is requested, regardless of where the request is coming from.

When to use it:

For lightweight, stateless services that perform a single operation.

For services that should not share state and need to be isolated from other consumers.

Examples: Simple utility classes, or a service that calculates a value and immediately returns it.

```cs
services.AddScoped<TService, TImplementation>()
```

Definition: A single instance of the service is created per scope. In a typical ASP.NET Core web application, the scope is usually the client's request (HTTP request).

This means that within one HTTP request, the service will be the same instance.

If the same client sends another request, a brand new instance will be created for that second request.

When to use it:

For services that need to maintain state during a single request but should not persist across requests.

This is the most common lifetime for services interacting with request-specific data, such as a database context (like DbContext in Entity Framework Core).

Examples: A service that coordinates multiple repository operations for a single transaction, or a service that holds request-specific user information.

```cs
services.AddSingleton<TService, TImplementation>()
```

Definition: A single instance of the service is created the first time it is requested (or when Startup.ConfigureServices runs, depending on the registration method) and that same instance is reused for every subsequent request for the entire lifetime of the application.

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

There is a microsoft library to which you can register your services. You can then ask a ServiceCollection to resolve services for you. You will reuse all the services.

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

With our Program.cs equal to this

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

```cs
/* Service locator who returns which services exist and are registered. */

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

## Repository pattern

Repository Pattern: An abstraction layer between your application logic and data access. It encapsulates the logic for retrieving and storing domain objects, providing a collection-like interface. The repository knows about your domain models and business logic.

In API, the view is the JSON response.

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
    var electronics = (Electronics)product;
    electronics.Price *= 0.8m;
}

// write this instead
if (product is Electronics electronis){
    electronics *=0.8m;
}
```
