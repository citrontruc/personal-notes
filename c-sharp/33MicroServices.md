# MicroServices

## Table of content

- [MicroServices](#microservices)
  - [Table of content](#table-of-content)
  - [Communications between microservices](#communications-between-microservices)
  - [Cool libraries](#cool-libraries)

## Communications between microservices

We want to avoid failure at all costs because they can lead to cascading failures and break down the whole system. That is why we need retry mechanisms such as under (retry with exponential backoff):

```cs
builder.Services.AddHttpClient<PropertyServiceClient>(client =>
{
    client.BaseAddress = new Uri("https://property-service:5001");
})
.AddStandardResilienceHandler(options =>
{
    options.Retry.MaxRetryAttempts = 3;
    options.Retry.BackoffType = DelayBackoffType.Exponential;
    options.Retry.Delay = TimeSpan.FromMilliseconds(500);
    
    options.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(10);
    options.CircuitBreaker.FailureRatio = 0.9;
    options.CircuitBreaker.MinimumThroughput = 5;
    
    options.AttemptTimeout.Timeout = TimeSpan.FromSeconds(2);
});
```

## Cool libraries

A quick example of how to model an app for microservices is this one: <https://learn.microsoft.com/en-us/azure/architecture/microservices/model/domain-analysis>.

Example git repository: <https://github.com/dotnet/eShop/tree/main>.

.Net Aspire, Polly for reliability or Dapr sidecars.
