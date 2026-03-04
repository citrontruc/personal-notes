# Proxy

## table of content

- [Proxy](#proxy)
  - [table of content](#table-of-content)
  - [Different types of proxy](#different-types-of-proxy)

## Different types of proxy

| Feature | Forward Proxy | Reverse Proxy |
| --- | --- | --- |
| Direction | Sits in front of clients (users). | Sits in front of web servers. |
| Primary Goal | Protects/anonymizes the client. | Protects/optimizes the backend servers. |
| Anonymity | Hides client IP from the internet. | Hides server IP from the client. |
| Traffic Control | Restricts client access to websites. | Balances traffic across multiple servers. |
| Caching | Caches external content for internal users. | Caches server content for external users. |
| SSL | Usually handles client-side encryption. | Performs SSL Termination for servers. |
| Common Use | Bypassing firewalls or monitoring usage. | Load balancing and security (WAF). |

A reverse proxy behaves kinda like an API Gateway. Difference is in the fact that they don't act at the same level and they don't have the same purpose. The role of the API gateway is to coordinate APIs and do a bit of verification in the process. For one user query, you may need multiple calls. The reverse proxy is donee th handle performance and security. It sits in the network / subnet.

| Feature | Reverse Proxy (e.g., NGINX, YARP) | API Gateway (e.g., Azure API Management, Ocelot) |
| --- | --- | --- |
| Focus | Traffic routing & performance. IP/Network level, Performance, Security. | API Lifecycle & business logic. Application/API level, Orchestration. |
| Granularity | Works at the host/path level. | Works at the endpoint/method level. |
| Auth | Basic (IP filtering, basic auth). | Complex (JWT validation, OAuth2, Scopes). |
| Transformation | Headers and basic body rewrites. | Request/Response mapping and protocol translation. |
| Billing/Usage | None. | Rate limiting, quotas, and monetization. |
| Aggregation | Not typical. | Aggregates multiple service calls into one response. |

YARP is a good option in dotnet for a reverse proxy & API Gateway

```cs
var builder = WebApplication.CreateBuilder(args);

// Add YARP services
// You need to have your configuration in the appsettings file. Add the requests to catch...
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();

// Map YARP routes
app.MapReverseProxy();

app.Run();
```
