# Application insights

## Terminology

Application insights monitors web pages/web service and background serivces, and sends everything to Azure Monitor. It monitors stuff like request rate, response time, dependeny rates, page views/load performance, user count, custom events/metrics

## Monitoring in ASP.NET

<https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net>

Note that in visual studio, it automatically updates the boilerplate. You just add the package, and it automatically starts monitoring.

## Custom events/metrics

<https://docs.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics>

```csharp
private TelemetryClient telemetry = new TelemetryClient();
TelemetryClient.Context.User.Id = "...";
TelemetryClient.Context.Device.Id = "...";
// Now, call the following in your code whenever an event with the name 'WinGame' happens
telemetry.TrackEvent("WinGame");
// Metrics work very similar. However, the value is not sent right away, but is aggregated
telemetry.GetMetric("CowsSold").TrackValue(42);

// You can also log traces, such as long messages or error stack traces
telemetry.TrackTrace("Slow response - database01");
```

---
Below is a bunch of boilerplate code

## Monitoring in .NET

First of all, you need to add Application insights SDK

**BOILERPLATE:**

.csproj

```xml
<ItemGroup>
    <PackageReference Include="Microsoft.ApplicationInsights" Version="2.12.1" />
</ItemGroup>
```

add this method to startup class

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // The following line enables Application Insights telemetry collection.
    services.AddApplicationInsightsTelemetry();

    // This code adds other services for your application.
    services.AddMvc();
}
```

appsetings.json

```json

{
    "ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    }
}
```

### Configuring the logger options

add this method to startup class

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // The following line enables Application Insights telemetry collection.
    services.AddOptions<ApplicationInsightsLoggerOptions>()
    .Configure(o => o.IncludeEventId = true)'

    // This code adds other services for your application.
    services.AddMvc();
}
```