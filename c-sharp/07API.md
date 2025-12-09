# API

## Table of content

- [API](#api)
  - [Table of content](#table-of-content)
  - [Avoid confusion in variable origin](#avoid-confusion-in-variable-origin)
  - [Swagger](#swagger)
  - [Example controller](#example-controller)
  - [Versioning](#versioning)
  - [Pagination](#pagination)
  - [Validation](#validation)

## Avoid confusion in variable origin

You can ask parameters [FromBody] or [FromQuery].

## Swagger

In order to have an automatic swagger for your application, you must add the swashbuckle library to your csproj. You can also use scalar to display a more modern swagger. You can't have both.

Very nice tutorial over here: <https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-8.0&viewFallbackFrom=aspnetcore-10.0&tabs=visual-studio-code>

You can use scalar to do your documentation: <https://www.mykolaaleksandrov.dev/posts/2025/11/scalar-api-documentation/>

tuto scalar : <https://darthpedro.net/2025/03/16/using-scalar-with-net9-webapi-projects/>.

## Example controller

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

namespace TodoApi.Controllers;

[Route("api/[controller]")]
[ApiController]
public class TodoItemsController : ControllerBase
{
    private readonly TodoContext _context;

    public TodoItemsController(TodoContext context)
    {
        _context = context;
    }

    // GET: api/TodoItems
    [HttpGet]
    public async Task<ActionResult<IEnumerable<TodoItemDTO>>> GetTodoItems()
    {
        return await _context.TodoItems
            .Select(x => ItemToDTO(x))
            .ToListAsync();
    }

    // GET: api/TodoItems/5
    // <snippet_GetByID>
    [HttpGet("{id}")]
    public async Task<ActionResult<TodoItemDTO>> GetTodoItem(long id)
    {
        var todoItem = await _context.TodoItems.FindAsync(id);

        if (todoItem == null)
        {
            return NotFound();
        }

        return ItemToDTO(todoItem);
    }
    // </snippet_GetByID>

    // PUT: api/TodoItems/5
    // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
    // <snippet_Update>
    [HttpPut("{id}")]
    public async Task<IActionResult> PutTodoItem(long id, TodoItemDTO todoDTO)
    {
        if (id != todoDTO.Id)
        {
            return BadRequest();
        }

        var todoItem = await _context.TodoItems.FindAsync(id);
        if (todoItem == null)
        {
            return NotFound();
        }

        todoItem.Name = todoDTO.Name;
        todoItem.IsComplete = todoDTO.IsComplete;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException) when (!TodoItemExists(id))
        {
            return NotFound();
        }

        return NoContent();
    }
    // </snippet_Update>

    // POST: api/TodoItems
    // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
    // <snippet_Create>
    [HttpPost]
    public async Task<ActionResult<TodoItemDTO>> PostTodoItem(TodoItemDTO todoDTO)
    {
        var todoItem = new TodoItem
        {
            IsComplete = todoDTO.IsComplete,
            Name = todoDTO.Name
        };

        _context.TodoItems.Add(todoItem);
        await _context.SaveChangesAsync();

        return CreatedAtAction(
            nameof(GetTodoItem),
            new { id = todoItem.Id },
            ItemToDTO(todoItem));
    }
    // </snippet_Create>

    // DELETE: api/TodoItems/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteTodoItem(long id)
    {
        var todoItem = await _context.TodoItems.FindAsync(id);
        if (todoItem == null)
        {
            return NotFound();
        }

        _context.TodoItems.Remove(todoItem);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool TodoItemExists(long id)
    {
        return _context.TodoItems.Any(e => e.Id == id);
    }

    private static TodoItemDTO ItemToDTO(TodoItem todoItem) =>
       new TodoItemDTO
       {
           Id = todoItem.Id,
           Name = todoItem.Name,
           IsComplete = todoItem.IsComplete
       };
}
```

## Versioning

Only do it when you add new stuff (you don't need it if you add optional fields or new endpoints). Put the number in the url path.

Use the Asp.Versioning.Mvc and configure the Dependency Injection

```cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Add API versioning
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
});

var app = builder.Build();

app.MapControllers();
app.Run();
```

You can also have versioning in your header. Do this in case of frequent change, meaningless version numbers or if you are openAI.

## Pagination

There are two main types of pagination: by page or by cursor. In both cases, you will take **pageSize** number of elements. Difference is in the fact that in page, you will cycle through pages and skip elements while in cursor pagination, you sort elements by id and take the ids bigger than your cursor.

Page is very easy to code but can be slow because elements you skip are still scanned. Problem is also that you have no idempotence (page orders is handled by the server). However, pages can be rebuilt from any scenario ==> Easy to do pagination on sorted data.

Cursor is idempotent + Since you give an id, you search by id which is much faster than skipping rows. Problem is that it depends on ids which are handled by the server (you don't know what ids are meant to be). Cursor does not solve some complex sorting patterns that pages can handle.

Cursor pagination translates to the following SQL query.

```sql
SELECT u.id, u.date, u.note, u.user_id
FROM user_notes AS u
WHERE u.date < @date OR (u.date = @date AND u.id <= @lastId)
ORDER BY u.date DESC, u.id DESC
LIMIT @limit;
```

It is possible to passs down a cursor to the user. Here is a way to encode a cursor and give it to the user:

```cs
using Microsoft.AspNetCore.Authentication; // For Base64UrlTextEncoder

public sealed record Cursor(DateOnly Date, Guid LastId)
{
    public static string Encode(DateOnly date, string lastId)
    {
        var cursor = new Cursor(date, lastId);
        string json = JsonSerializer.Serialize(cursor);
        return Base64UrlTextEncoder.Encode(Encoding.UTF8.GetBytes(json));
    }

    public static Cursor? Decode(string? cursor)
    {
        if (string.IsNullOrWhiteSpace(cursor))
        {
            return null;
        }

        try
        {
            string json = Encoding.UTF8.GetString(Base64UrlTextEncoder.Decode(cursor));
            return JsonSerializer.Deserialize<Cursor>(json);
        }
        catch
        {
            return null;
        }
    }
}
```

## Validation

You want to make sure that the values that get posted are at the very least checked. There is a validation feature in Dotnet 10: <https://www.nikolatech.net/blogs/minimal-api-validation-in-aspnet-core>

Just add a validation service to your webAppBuilder

```cs
builder.Services.AddValidation();
```

You also need to add to the csproj file the following information:

```xml
<PropertyGroup>
<InterceptorsNamespaces>$(InterceptorsNamespaces);Microsoft.AspNetCore.Http.Validation.Generated</InterceptorsNamespaces>
</PropertyGroup>
```

We can now have data validation through a validation class:

```cs
public sealed record CreateProductRequest(
    string Name,
    string Description,
    decimal Price) : IValidatableObject
{
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (string.IsNullOrWhiteSpace(Name))
        {
            yield return new ValidationResult("Name should not be empty.", [nameof(Name)]);
        }
        if (string.IsNullOrWhiteSpace(Description))
        {
            yield return new ValidationResult("Description should not be empty.", [nameof(Description)]);
        }
        if (Price < 0.01m || Price > 10.0m)
        {
            yield return new ValidationResult("Price should be between 0.01 and 10.0.", [nameof(Price)]);
        }
    }
}
```

You can also add validation in requests:

```cs
public sealed record CreateProductRequest(
    [Required] string Name,
    [Required] string Description,
    [Range(0.01, 10.0)] decimal Price);
```
