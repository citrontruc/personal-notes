# Authentication & Authorization

## Table of content

- [Authentication \& Authorization](#authentication--authorization)
  - [Table of content](#table-of-content)
  - [Adding auth middleware](#adding-auth-middleware)
    - [Code](#code)
    - [Configuration](#configuration)
  - [Create policies](#create-policies)

## Adding auth middleware

### Code

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

### Configuration

In order to specify how to do authentication, you must specidy the auth configuration in your appsettings.json file.

```json
{
  "Authentication": {
    "DefaultScheme":  "LocalAuthIssuer",
    "Schemes": {
      "Bearer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "dotnet-user-jwts"
      },
      "LocalAuthIssuer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "local-auth"
      }
    }
  }
}
```

The ValidAudiences don't have to be urls but can be any  user role. The valid issuer can be a lot of things, for example, aws, azure or other: <https://login.microsoftonline.com/tenant-id/v2.0>. dotnet-user-jwts is the local jwt token issuer.

You can have multiple issuers if you allow SSO, if you are doing ddevelopment or if you are doing a migration between two security issuers.

Full configuration for your authentication needs to be in your configuration file.

In order to create a security token, run the following command:

```bash
dotnet user-jwts create
```

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
