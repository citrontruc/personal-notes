# Communications with server

## Table of content

- [Communications with server](#communications-with-server)
  - [Table of content](#table-of-content)
  - [HttpClient](#httpclient)
  - [Server sent events](#server-sent-events)

## HttpClient

HttpClient  are not disposed of by default so make sure to either mutualize them or dispose of them. Since HTTPclients are meant to have a long lifetime, mutualizing sounds like the optimal solution.

There is an HttpClientFactory library that can handle creating an disposing of HttpClient. <https://code-maze.com/using-httpclientfactory-in-asp-net-core-applications/>

## Server sent events

Principle is simple: client initiates a connection to the server and the server sends updates when they happen. Connection is kept open.

You will find under an implementation of an SSE that returns random weather forecasts. One of the challenges is to reconnect on connection lost. Example right underneath:

```cs
// To check response use: curl http://localhost:5285/weatherforecast
app.MapGet("/weatherforecast", async (HttpRequest request, CancellationToken cancellationToken) =>
{
    var lastEventId =
        request.Headers.TryGetValue("Last-Event-ID", out var value)
            ? value.ToString()
            : null;

    if (lastEventId is not null)
    {
        // Handle reconnection logic if needed
        // To reconnect use:
        //     curl -H "Last-Event-ID: yyyy-MM-dd'T'HH:mm:ss.fffffffK" http://localhost:5285/weatherforecast
    }

    return TypedResults.ServerSentEvents(GetWeatherForecast(cancellationToken));
})

async IAsyncEnumerable<SseItem<WeatherForecast>> GetWeatherForecast(
    [EnumeratorCancellation] CancellationToken cancellationToken)
{
    while (!cancellationToken.IsCancellationRequested)
    {
        var eventId = DateTimeOffset.UtcNow.ToString("O");

        var forecast = new WeatherForecast(
            DateOnly.FromDateTime(DateTime.Now),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]);

        yield return new SseItem<WeatherForecast>(forecast, eventType: "weatherForecast")
        {
            EventId = eventId,
            ReconnectionInterval = TimeSpan.FromMilliseconds(500)
        };

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
}
```
