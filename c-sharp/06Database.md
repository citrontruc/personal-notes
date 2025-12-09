# Database

## Table of content

- [Database](#database)
  - [Table of content](#table-of-content)
  - [Migrations](#migrations)
  - [Filter data](#filter-data)
  - [Index](#index)
  - [Tracking](#tracking)
  - [Projections](#projections)
  - [Loading data](#loading-data)
    - [Eager loading](#eager-loading)
    - [Lazy loading](#lazy-loading)
    - [Explicit loading](#explicit-loading)
  - [Add rollback to operations](#add-rollback-to-operations)
  - [Locks](#locks)
  - [Dispose of DbContext](#dispose-of-dbcontext)

## Migrations

**WARNING**: Use EF Core migrations to manage schema changes.

Apply changes to data model.

```bash
dotnet ef migrations add InitialCreate
```

If we change the object schemas, we change our database accordingly by using:

```bash
dotnet ef migrations add AddBlogCreatedTimestamp
dotnet ef database update
```

If your database is in the cloud / elsewhere, you can update your migrations with the connection string.

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
