# Authentication & Authorization

In minimal APIs, authentication and authorization are handled through a service that is used in the middleware.

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```
