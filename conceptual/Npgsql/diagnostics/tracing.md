# Tracing with OpenTelemetry (experimental)

> [!NOTE]
> Support for tracing via OpenTelemetry has been introduced in Npgsql 6.0.
>
> [The OpenTelemetry specifications for database tracing](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/semantic_conventions/database.md) are currently experimental, so Npgsql's support may change in upcoming releases.

[OpenTelemetry](https://opentelemetry.io/) is a widely-adopted framework for distributed observability across many languages and components; its tracing standards allow applications and libraries to emit information on activities and events, which can be exported by the application, stored and analyzed. Activities typically have start and end times, and can encompass other activities recursively; this allows you to analyze e.g. exactly how much time was spent in the database when handling a certain HTTP call.

To make Npgsql emit tracing data, reference the [Npgsql.OpenTelemetry](https://www.nuget.org/packages/Npgsql.OpenTelemetry) NuGet package from your application, and set up tracing as follows:

```c#
using var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService("npgsql-tester"))
    .SetSampler(new AlwaysOnSampler())
    // This optional activates tracing for your application, if you trace your own activities:
    .AddSource("MyApp")
    // This activates up Npgsql's tracing:
    .AddNpgsql()
    // This prints tracing data to the console:
    .AddConsoleExporter()
    .Build();
```

Once this is done, you should start seeing Npgsql trace data appearing in your application's console. At this point, you can look into exporting your trace data to a more useful destination: systems such as [Zipkin](https://zipkin.io/) or [Jaeger](https://www.jaegertracing.io/) can efficiently collect and store your data, and provide user interfaces for querying and exploring it. Setting these up in your application is quite easy - simply replace the console exporter with the appropriate exporter for the chosen system.

For example, Zipkin visualizes traces in the following way:

![Zipkin UI Sample](/img/zipkin.png)

In this trace, the Npgsql query (to database testdb) took around 800ms, and was nested inside the application's `work1` activity, which also had another unrelated `subtask1`. This allows understanding the relationships between the different activities, and where time is being spent.

### Enrich Options

Enrich actions on the tracing options allow activities created by Npgsql to be enriched with additional information from the raw object relating to the activity, or on any exception.
The action is called only when `activity.IsAllDataRequested` is `true`.

#### `EnrichCommandExecution`

This action's parameters contain the activity itself (which can be enriched), the name of the event, and either the `NpgsqlCommand` or a tuple also containing an exception, depending on the event name:

For event name "OnStartActivity", the actual object will be `NpgsqlCommand`.

For event name "OnFirstResponse", the actual object will be `NpgsqlCommand`.

For event name "OnStopActivity", the actual object will be `NpgsqlCommand`.

For event name "OnException", the actual object will be `ValueTuple<NpgsqlCommand, Exception>`.

Example:

```csharp
using System;
using Npgsql;
using OpenTelemetry;
using OpenTelemetry.Trace;

var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .AddNpgsql(options => options.EnrichCommandExecution
        = (activity, eventName, rawObject) =>
        {
            switch (eventName, rawObject)
            {
                case ("OnStartActivity", NpgsqlCommand command):
                    activity.SetTag("command.type", command.CommandType);
                    break;
                case ("OnFirstResponse", NpgsqlCommand command):
                    activity.SetTag("received-first-response", DateTime.UtcNow);
                    break;
                case ("OnStopActivity", NpgsqlCommand command):
                    activity.SetTag("command.type", command.CommandType);
                    break;
                case ("OnException", (NpgsqlCommand command, Exception exception)):
                    activity.SetTag("stackTrace", exception.StackTrace);
                    break;
            }
        }).Build();
```

### Record Options

Recording of an `ActivityEvent` can be disabled for exceptions or the point at which a first response is received, by setting the corresponding flags on the options.
Note that even when disabled, the corresponding Enrich invocations will still occur ("OnException" and "OnFirstResponse" respectively, see above).

Example:

```csharp
using System;
using Npgsql;
using OpenTelemetry;
using OpenTelemetry.Trace;

var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .AddNpgsql(options => 
        {
            options.RecordCommandExecutionException = false; // Default = true
            options.RecordCommandExecutionFirstResponse = false; // Default = true
        }).Build();
```