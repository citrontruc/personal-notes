# Authentication & Authorization

## Table of content

- [Authentication \& Authorization](#authentication--authorization)
  - [Table of content](#table-of-content)
  - [Adding auth middleware](#adding-auth-middleware)
    - [Code](#code)
    - [Configuration](#configuration)
  - [Create policies](#create-policies)
  - [JWT token authentication](#jwt-token-authentication)

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
builder.Services.AddAuthentication("Bearer").AddJwtBearer(); // Bearer is the default scheme. You don't need to write it.
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseCors();
app.UseAuthentication(); 
app.UseAuthorization();

app.MapGet("/", () => "Hello World!"); // order of declaration matters so check that you have authentication before endpoint declaration.
app.Run();
```

When we add the JWT bearer, we can pass options specifying what we want to validate and how we want to do it.

```cs
builder.Services.AddAuthentication("Bearer")
  .AddJwtBearer(options =>
  {
      options.TokenValidationParameters = new()
      {
          ValidateIssuer = true,
          ValidateAudience = true,
          ValidateIssuerSigningKey = true,
          ValidIssuer = builder.Configuration["Authentication:Issuer"],
          ValidAudience = builder.Configuration["Authentication:Audience"],
          IssuerSigningKey = new SymmetricSecurityKey(
              Convert.FromBase64String(builder.Configuration["Authentication:SecretForKey"]))
      };
  }
);
// Validate the the user must have a valid token with the correct information.
// Expiration time is automatically verified. You don't need to specify it.

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("MustBeFromAntwerp", policy =>
    {
        policy.RequireAuthenticatedUser();
        policy.RequireClaim("city", "Antwerp");
    });
});
```

Use the [Authorize] attribute for the endpoints you want to authenticate. We can also specify in the attribute what we need with for example: [Authorize(Policy = "MustBeFromAntwerp")].

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
        "ValidIssuer": "dotnet-user-jwts" // Library to simulate an external token authority.
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

We can also add all sorts of information on our token. In order to have the full list of possible arguments, try dotnet user-jwts create --help

```bash
dotnet user-jwts create --issuer https://localhost:7169 --audience audienceInfo --claim "value=expectedValue"
```

In order to check the key with which the jwt was signed, use the following command. You can then add this key to your dev appsettings.

```bash
dotnet user-jwts key --issuer https://localhost:7169
```

To get the list of all the tokens that you have used, you can use the following command.

```bash
dotnet user-jwts list
```

```bash
dotnet user-jwts print token-id
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

## JWT token authentication

Our authentication class is the following.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace CityInfo.API.Controllers
{
    [Route("api/authentication")]
    [ApiController]
    public class AuthenticationController : ControllerBase
    {
        private readonly IConfiguration _configuration;

        // we won't use this outside of this class, so we can scope it to this namespace
        // Normally, we would put it in its own folder.
        public class AuthenticationRequestBody
        {
            public string? UserName { get; set; }
            public string? Password { get; set; } // Don't use plain password, use a hash.
        }

        private class CityInfoUser
        {
            public int UserId { get; set; }
            public string UserName { get; set; }
            public string FirstName { get; set; }
            public string LastName { get; set; }
            public string City { get; set; }

            public CityInfoUser(
                int userId, 
                string userName, 
                string firstName, 
                string lastName, 
                string city)
            {
                UserId = userId;
                UserName = userName;
                FirstName = firstName;
                LastName = lastName;
                City = city;
            }

        }

        public AuthenticationController(IConfiguration configuration)
        {
            _configuration = configuration ?? 
                throw new ArgumentNullException(nameof(configuration));
        }

        [HttpPost("authenticate")]
        public ActionResult<string> Authenticate(
            AuthenticationRequestBody authenticationRequestBody)
        {  
            // Step 1: validate the username/password
            var user = ValidateUserCredentials(
                authenticationRequestBody.UserName,
                authenticationRequestBody.Password);

            if (user == null)
            {
                return Unauthorized();
            }

            // Step 2: create a token
            var securityKey = new SymmetricSecurityKey(
                Convert.FromBase64String(_configuration["Authentication:SecretForKey"]));
            var signingCredentials = new SigningCredentials(
                securityKey, SecurityAlgorithms.HmacSha256);
             
            var claimsForToken = new List<Claim>();
            claimsForToken.Add(new Claim("sub", user.UserId.ToString()));
            claimsForToken.Add(new Claim("given_name", user.FirstName));
            claimsForToken.Add(new Claim("family_name", user.LastName));
            claimsForToken.Add(new Claim("city", user.City));
             
            var jwtSecurityToken = new JwtSecurityToken(
                _configuration["Authentication:Issuer"], // Issuer of the token
                _configuration["Authentication:Audience"], // Audience (access rights)
                claimsForToken, // For user
                DateTime.UtcNow, // Validity start
                DateTime.UtcNow.AddHours(1), // End of validity
                signingCredentials);

            var tokenToReturn = new JwtSecurityTokenHandler()
               .WriteToken(jwtSecurityToken);

            return Ok(tokenToReturn);
        }

        private CityInfoUser ValidateUserCredentials(string? userName, string? password)
        {
            // we don't have a user DB or table.  If you have, check the passed-through
            // username/password against what's stored in the database.
            //
            // For demo purposes, we assume the credentials are valid

            // return a new CityInfoUser (values would normally come from your user DB/table)
            // We don't handle the creation of new credentials.
            return new CityInfoUser(
                1,
                userName ?? "",
                "Kevin",
                "Dockx",
                "Antwerp");

        }
    }
}
```
