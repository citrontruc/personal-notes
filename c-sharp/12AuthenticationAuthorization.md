# Authentication & Authorization

In minimal APIs, authentication and authorization are handled through a service that is used in the middleware.

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication();
// builder.Services.AddAuthentication().AddJwtBearer().AddJwtBearer("LocalAuthIssuer");; // You can add different types of authentication and different configurations of authentication.
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```

There are multiple services that can be run. Be very careful about the order of operations and that you have all the operations in place:

```cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors();
builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseCors();
app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/", () => "Hello World!");
app.Run();
```

Full configuration for your authentication needs to be in your configuration file.

## Create policies

If you want to have something more granular, you can add your own policy depending on access rights:

```cs
using Microsoft.Identity.Web;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorizationBuilder()
  .AddPolicy("admin_greetings", policy =>
        policy
            .RequireRole("admin")
            .RequireClaim("scope", "greetings_api"));

var app = builder.Build();

app.MapGet("/hello", () => "Hello world!")
  .RequireAuthorization("admin_greetings");

app.Run();
```

When using jwt locally, you can use the dotnet user-jwts to create tokens with a specific scope: <https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-10.0>
