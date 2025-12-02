# Database

**WARNING**: Use EF Core migrations to manage schema changes.

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

## Eager loading

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
