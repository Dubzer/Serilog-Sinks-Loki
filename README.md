# Serilog.Sinks.Loki

This is a Serilog Sink for Grafana's new [Loki Log Aggregator](https://github.com/grafana/loki).

Note: It's still really early work in progress.

## Current Features:

- Formats and batches log entries to Loki via HTTP
- Ability to provide global Loki log labels

Coming soon:

- Ability to provide contextual log labels
- Write logs to disk in the correct format to send via Promtail
- Send logs to Loki via HTTP using Snappy compression

## Basic Example:

```csharp
Log.Logger = new LoggerConfiguration()
        .MinimumLevel.Information()
        .Enrich.FromLogContext()
        .WriteTo.LokiHttp("http://localhost:3100/api/prom/push")
        .CreateLogger();

var exception = new {Message = ex.Message, StackTrace = ex.StackTrace};
Log.Error(exception);

var position = new { Latitude = 25, Longitude = 134 };
var elapsedMs = 34;
Log.Information("Message processed {@Position} in {Elapsed:000} ms.", position, elapsedMs);

Log.CloseAndFlush();
```

### Adding global labels

Loki indexes and groups log streams using labels, in Serilog.Sinks.Loki you can attach labels to all log entries by passing an implementation `ILogLabelProvider` to the `WriteTo.LokiHttp(..)` configuratino method. This is idea for labels such as instance IDs, environments and application names:

```csharp
public class LogLabelProvider : ILogLabelProvider {

    public IList<LokiLabel> GetLabels()
    {
        return new List<LokiLabel>
        {
            new LokiLabel { Key = "app", Value = "demoapp" },
            new LokiLabel { Key = "environment", Value = "production" }
        };
    }

}
```
```csharp
var log = new LoggerConfiguration()
        .MinimumLevel.Verbose()
        .Enrich.FromLogContext()
        .WriteTo.LokiHttp("http://localhost:3100/api/prom/push", new LogLabelProvider())
        .CreateLogger();
```

### Custom HTTP Client

Serilog.Loki.Sink is built on top of the popular [Serilog.Sinks.Http](https://github.com/FantasticFiasco/serilog-sinks-http) library to post log entries to Loki. With this in mind you can you can extend the default HttpClient (`LokiHttpClient`), or replace it entirely by implementing `IHttpClient`.

```csharp
// ExampleHttpClient.cs

public class ExampleHttpClient : LokiHttpClient
{
    public override Task<HttpResponseMessage> PostAsync(string requestUri, HttpContent content)
    {
        return base.PostAsync(requestUri, content);
    }
}
```
```csharp
// Usage

var log = new LoggerConfiguration()
        .MinimumLevel.Verbose()
        .Enrich.FromLogContext()
        .WriteTo.LokiHttp("http://localhost:3100/api/prom/push", new LogLabelProvider(), new ExampleHttpClient())
        .CreateLogger();
```
