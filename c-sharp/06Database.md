# Database

## Table of content

- [Database](#database)
  - [Table of content](#table-of-content)
  - [Notes on performances](#notes-on-performances)
    - [Tip 1) Reuse connection when possible](#tip-1-reuse-connection-when-possible)
    - [Tip 2) regroup queries whenever possible \& entity framework](#tip-2-regroup-queries-whenever-possible--entity-framework)
    - [Tip 3) Transform data](#tip-3-transform-data)
    - [Tip 4) Deploy in multiple regions](#tip-4-deploy-in-multiple-regions)
    - [Bulk insert](#bulk-insert)
  - [Migrations](#migrations)
    - [WARNING EnsureCreated](#warning-ensurecreated)
  - [Filter data](#filter-data)
  - [Index](#index)
  - [Tracking](#tracking)
  - [Projections](#projections)
  - [Loading data](#loading-data)
    - [Eager loading](#eager-loading)
    - [Lazy loading](#lazy-loading)
    - [Explicit loading](#explicit-loading)
  - [Add rollback to operations](#add-rollback-to-operations)
  - [Split Queries](#split-queries)
  - [Locks](#locks)
  - [Dispose of DbContext](#dispose-of-dbcontext)
  - [Global Query Filters](#global-query-filters)
  - [Useful scripts](#useful-scripts)
    - [Count employees](#count-employees)
  - [Examples](#examples)

## Notes on performances

DbContext serves as an ORM but you can bypass it and directly send an SQL query. Can be useful for some optimization cases.

### Tip 1) Reuse connection when possible

Opening and closing connections takes time. Avoid to open and close them. Reuse them.

```cs
var cmdText = @"
    insert into dbo.Customers (Id, FirstName, LastName, Street, City, State, PhoneNumber, EmailAddress)
    values (@Id, @FirstName, @LastName, @Street, @City, @State, @PhoneNumber, @EmailAddress)";

// We create a connection and use it for all our customers.
using (var connection = new SqlConnection(connectionString))
{
    foreach (var customer in customers)
    {
        var command = new SqlCommand(cmdText, connection);
        command.Parameters.AddWithValue("@Id", customer.Id);
        command.Parameters.AddWithValue("@FirstName", customer.FirstName);
        command.Parameters.AddWithValue("@LastName", customer.LastName);
        command.Parameters.AddWithValue("@Street", customer.Street);
        command.Parameters.AddWithValue("@City", customer.City);
        command.Parameters.AddWithValue("@State", customer.State);
        command.Parameters.AddWithValue("@PhoneNumber", customer.PhoneNumber);
        command.Parameters.AddWithValue("@EmailAddress", customer.EmailAddress);
 
        connection.Open();
        command.ExecuteNonQuery();
    }
}
```

### Tip 2) regroup queries whenever possible & entity framework

Instead of having one SQL query for each row, we could have one giant sql query which encompasses all our users. Avoid doing this too much because it is ugly but you could do it.

```cs
var cmdText = customers.Aggregate(
    new StringBuilder(),
    (sb, customer) => sb.AppendLine(@$"
        insert into dbo.Customers (Id, FirstName, LastName, Street, City, State, PhoneNumber, EmailAddress)
        values('{customer.Id}', '{customer.FirstName}', '{customer.LastName}', '{customer.Street}', '{customer.City}', '{customer.State}', '{customer.PhoneNumber}', '{customer.EmailAddress}')")
);
 
using (var connection = new SqlConnection(connectionString))
{
    var command = new SqlCommand(cmdText.ToString(), connection);
    connection.Open();
    command.ExecuteNonQuery();
}
```

it is much better to use the entity framework

```cs
using (var context = new CustomersContext())
{
    context.Customers.AddRange(customers);
    context.SaveChanges();
}
```

In this case, an optimized SQL query is generated automatically.

### Tip 3) Transform data

By transforming data into a data table, we make them easier to insert in an sql table.

```cs
DataTable GetUsersDataTable()
{
    var dataTable = new DataTable();

    dataTable.Columns.Add(nameof(User.Email), typeof(string));
    dataTable.Columns.Add(nameof(User.FirstName), typeof(string));
    dataTable.Columns.Add(nameof(User.LastName), typeof(string));
    dataTable.Columns.Add(nameof(User.PhoneNumber), typeof(string));

    foreach (var user in GetUsers())
    {
        dataTable.Rows.Add(
            user.Email, user.FirstName, user.LastName, user.PhoneNumber);
    }

    return dataTable;
}
```

### Tip 4) Deploy in multiple regions

Avoid latency.

### Bulk insert

Method to insert large volumes of data real fast. There are two main ways to do this, the first is SqlBulkCopy. Problem is that it only works for SQl.

```cs
using (var copy = new SqlBulkCopy(connectionString))
{
    copy.DestinationTableName = "dbo.Customers";
    // Add mappings so that the column order doesn't matter
    copy.ColumnMappings.Add(nameof(Customer.Id), "Id");
    copy.ColumnMappings.Add(nameof(Customer.FirstName), "FirstName");
    copy.ColumnMappings.Add(nameof(Customer.LastName), "LastName");
    copy.ColumnMappings.Add(nameof(Customer.Street), "Street");
    copy.ColumnMappings.Add(nameof(Customer.City), "City");
    copy.ColumnMappings.Add(nameof(Customer.State), "State");
    copy.ColumnMappings.Add(nameof(Customer.PhoneNumber), "PhoneNumber");
    copy.ColumnMappings.Add(nameof(Customer.EmailAddress), "EmailAddress");
 
    copy.WriteToServer(ToDataTable(customers));
}
```

Ef core sample with the BulkInsertAsync method:

```cs
using var context = new ApplicationDbContext();

await context.BulkInsertAsync(GetUsers());
```

BulkInsert has a few nifty options for optimization. You can have InsertIfNotExist to avoid inserting data that already exists. You have BulkUpdate, BulkDelete and other nice methods. Not just insert.

## Migrations

**WARNING**: Use EF Core migrations to manage schema changes.

Apply changes to data model. **WARNING**: dotnet ef libraries are not backward compatible. Specify which version you want depending on your project.

```bash
dotnet ef migrations add InitialCreate
```

If we change the object schemas, we change our database accordingly by using:

```bash
dotnet ef migrations add AddBlogCreatedTimestamp
dotnet ef database update
```

If your database is in the cloud / elsewhere, you can update your migrations with the connection string.

To view migrations, type:

```cs
dotnet ef migrations list
```

If you want to go back to an earlier migration, just type:

```bash
dotnet ef database update <PreviousMigrationName>
```

and then update migrations.

### WARNING EnsureCreated

When you have migrations, don't create database with EnsureCreated. The following code will conflict with migrations.

```cs
// Create database if it doesn't exist. Don't do that, this is AI BS
//using (var scope = app.Services.CreateScope())
//{
    //var context = scope.ServiceProvider.GetRequiredService<MusicDbContext>();
    // context.Database.EnsureCreated(); // Don't do that, that is a very bad practice. Use migrations.
//}
```

## Filter data

Pagination + filters

```cs
// Use paging to select fixed number of records
int pageSize = 50;
int pageNumber = 1;

var books = context.Books
    .AsNoTracking()
    .OrderBy(p => p.Title)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToList();
```

When using filters, you can use the .Contains method in order to filter on a column. Problem is if you have a lot of values to filter on (example: .Where(p => productIDs.Contains(p.ID)) with a list of 10_000 ids), you end up with a giant query that takes time or can even make your system crash (SQL has a parameter limit).

If you have a lot of values to retrieve, see bulk retrieval.

## Index

In order to accelerate research based on a specific field, use indexes to speed up search:

```cs
public class BookConfiguration : IEntityTypeConfiguration<Book>
{
    public void Configure(EntityTypeBuilder<Book> builder)
    {
        builder.ToTable("books");
        builder.HasKey(x => x.Id);
        builder.HasIndex(x => x.Title);
    }
}
```

Index the most used data on the most searched on columns.

## Tracking

By default, EF core tracks all queried entities even if they are not modified. That is unnecessary if you just do read operations ==> In order to avoid problems with tracking and speed up operations use .AsNoTracking()

```cs
var orders = await context.Orders
  .AsNoTracking()
  .Include(o => o.Customer)
  .ToListAsync();
```

Don't do this if you plan to modify data.

## Projections

In order to avoid fetching too many columns, use projections to use the right amount of columns

```cs
// Fetching only needed columns
var book = await context.Books
    .Where(b => b.Id == id)
    .Select(b => new BooksPreviewResponse
    {
        Title = b.Title,
        Author = b.Author.Name,
        Year = b.Year
    })
    .FirstOrDefaultAsync(cancellationToken);
```

## Loading data

### Eager loading

Tu fais ta requête et tu loades dans la foulée.

Eager loading is when you query one type of entity and immediately load related entities as part of it.

Include and ThenIndlude lets you specify which data to include. You can have multiple levels. **HOWEVER** Be careful!

```cs
// Include and ThenInclude are for eager loading,
// they can lead to performance issues if not used carefully
var book = await context.Books
    .AsNoTracking()
    .Include(b => b.Author)
        .ThenInclude(a => a.Publisher)
    .ToListAsync();

// You can use filters in Include and ThenInclude methods
// to limit data that is loaded together with the main entity
var authors = await context.Authors
    .Include(a => a.Books.Where(b => b.Year >= 2023))
    .ToListAsync();

using (var context = new BloggingContext())
{
    var filteredBlogs = await context.Blogs
        .Include(
            blog => blog.Posts
                .Where(post => post.BlogId == 1)
                .OrderByDescending(post => post.Title)
                .Take(5))
        .ToListAsync();
}

// You can do weird stuff.
using (var context = new BloggingContext())
{
    var blogs = await context.Blogs
        .Include(blog => blog.Posts)
        .ThenInclude(post => post.Author)
        .ThenInclude(author => author.Photo)
        .Include(blog => blog.Owner)
        .ThenInclude(owner => owner.Photo)
        .ToListAsync();
}
```

### Lazy loading

Les données sont chargées quand elles sont sollicitées. Desavantage : tu ne contrôles pas vraiment.

```cs
// Startup / DbContext options
optionsBuilder
    .UseLazyLoadingProxies()
    .UseSqlServer(connectionString);

public class User
{
    public int Id { get; set; }
    public virtual List<Post> Posts { get; set; }  // virtual for lazy loading
}

var user = await context.Users.FirstAsync();

// triggers a separate SQL query only when accessed
var posts = user.Posts;
```

### Explicit loading

You manually request loading of related data.

```cs
var user = await context.Users.FirstAsync();

// explicitly load the relationship later
await context.Entry(user)
    .Collection(u => u.Posts)
    .LoadAsync();

// now user.Posts is populated, but only because you asked for it
```

## Add rollback to operations

We gotta respect those ACID properties!

```cs
using var transaction = await _context.Database.BeginTransactionAsync();
try
{
    await context.Books
        .Where(b => b.Id == update.BookId)
        .ExecuteUpdateAsync(b => b
        .SetProperty(book => book.Title, book => update.NewTitle)
        .SetProperty(book => book.UpdatedAtUtc, book => DateTime.UtcNow));

    await context.Authors.AddAsync(newAuthor);
    await context.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch (Exception)
{
    // Now you can rollback both operations
    await transaction.RollbackAsync();
    throw;
}
```

## Split Queries

Imagine that you need to join multiple data together and then filter the result. By default you will have everything in a single query which can be memory intensive. A solution is to use "AsSplitquery" which will apply filters first on each table and then merge everything ==> You have multiple small queries.

```cs
// Before
var project = await _context.Projects
    .Include(p => p.Tasks)
    .Include(p => p.Contributors)
    .FirstOrDefaultAsync(p => p.Id == projectId);

// After
var project = await _context.Projects
    .AsSplitQuery() // <--- The Magic Trigger
    .Include(p => p.Tasks)
    .Include(p => p.Contributors)
    .FirstOrDefaultAsync(p => p.Id == projectId);
```

## Locks

Most db handle locking themselves and you don't have to set up your own locks.

You can however have concurrent calls. In which case, handle them wisely. They can happen mostly in updates:

```cs
modelBuilder.Entity<Book>()
    .Property<byte[]>("Version")
    .IsRowVersion();

var book = await context.Books
    .FirstOrDefaultAsync(b => b.Id == update.BookId);

book.Title = update.NewTitle;
book.Year = update.NewYear;
book.UpdatedAtUtc = DateTime.UtcNow;

try
{
    await _context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{} // Decide what to do here: for example, show error to the user
```

## Dispose of DbContext

Be careful with the dbcontext, if you create one and never dispose of it, you can have a memory leak.

```cs
// BAD :
private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

// 🚫 BAD CODE: Memory Leak, DbContext is not disposed
public async Task StartAsync(CancellationToken cancellationToken)
{
    var context = await _contextFactory.CreateDbContextAsync();
    var books = await context.Books.ToListAsync();
}

// GOOD :
private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

// ✅ DbContext is correctly disposed
public async Task StartAsync(CancellationToken cancellationToken)
{
    using var context = await _contextFactory.CreateDbContextAsync();
    var books = await context.Books.ToListAsync();
}
```

## Global Query Filters

In your modelBuilder, add a query filter to avoid inappropriate data fro coming up:

```cs
modelBuilder
   .Entity<Order>()
   .HasQueryFilter(order => !order.IsDeleted);
```

If you need to do a request without it, you can manually dipose of your query filter

```cs
dbContext
   .Orders
   .IgnoreQueryFilters()
   .Where(order => order.Id == orderId)
   .FirstOrDefault();
```

Careful: when adding query filters, you make your requests less clear.

Query filters are mostly used when you want to filter on a certain tenant value and make sure we retrieve only the data for our client.

```cs
public class ApplicationDbContext : DbContext
{
    private readonly Guid? _currentTenantId;
    
    public DbSet<Author> Authors { get; set; } = default!;
    
    public DbSet<Book> Books { get; set; } = default!;
    
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options,
        ITenantService tenantService) : base(options)
    {
        _currentTenantId = tenantService.GetCurrentTenantId();
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasQueryFilter(x => x.TenantId == _currentTenantId);
        
        base.OnModelCreating(modelBuilder);
        
        // Rest of the code remains unchanged
    }
}

// And We also need a class to retrieve the tenant id from the request headers.

public interface ITenantService
{
    Guid? GetCurrentTenantId();
}

public class TenantService : ITenantService
{
    private readonly Guid? _currentTenantId;
    
    public TenantService(IHttpContextAccessor accessor)
    {
        var headers = accessor.HttpContext?.Request.Headers;

        _currentTenantId = headers.TryGetValue("Tenant-Id", out var value) is true
            ? Guid.Parse(value.ToString())
            : null;
    }

    public Guid? GetCurrentTenantId() => _currentTenantId;
}
```

## Useful scripts

### Count employees

A little script to get the number of employees from an SQL database.

```cs
// return number of employees from database
public int GetEmployeesCount(string connectionString){
    using var connection = new SqlConnection(connectionString);
    connection.Open();

    using var command = new SqlCommand("Select COUNT(Id) from Employees", connection);

    int employeesCount = command.ExecuteNonQuery();
    return employeesCount;
}
```

## Examples

Example complete DbContext with a query filter.

```cs
public class ApplicationDbContext : DbContext
{
    public DbSet<Author> Authors { get; set; } = default!;
    
    public DbSet<Book> Books { get; set; } = default!;
    
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>()
            .HasQueryFilter(x => !x.IsDeleted);
        
        base.OnModelCreating(modelBuilder);
        
        modelBuilder.Entity<Author>(entity =>
        {
            entity.ToTable("authors");

            entity.HasKey(x => x.Id);
            entity.HasIndex(x => x.Name);
            
            entity.Property(x => x.Id).IsRequired();
            entity.Property(x => x.Name).IsRequired();
            entity.Property(x => x.Country).IsRequired();
            
            entity.HasMany(x => x.Books)
                .WithOne(x => x.Author);
        });
        
        modelBuilder.Entity<Book>(entity =>
        {
            entity.ToTable("books");

            entity.HasKey(x => x.Id);
            entity.HasIndex(x => x.Title);
            
            entity.Property(x => x.Id).IsRequired();
            entity.Property(x => x.Title).IsRequired();
            entity.Property(x => x.Year).IsRequired();
            entity.Property(x => x.IsDeleted).IsRequired();

            entity.HasOne(x => x.Author)
                .WithMany(x => x.Books);
        });
    }
}
```
