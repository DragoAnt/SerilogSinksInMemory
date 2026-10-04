# Sink reference

Creating the logger, isolating tests, snapshots and log levels for `DragoAnt.Serilog.Sinks.InMemory`. Back to the [README](../README.md).

## Clearing log events between tests

Depending on your test framework and test setup you may want to ensure that the log events captured by the `InMemorySink` are cleared so tests
are not interfering with eachother. To enable this, the `InMemorySink` implements the [`IDisposable`](https://docs.microsoft.com/en-us/dotnet/api/system.idisposable?view=netstandard-2.0) interface.
When `Dispose()` is called the `LogEvents` collection is cleared.

It will depend on the test framework or your test if you need this feature. With xUnit this feature is not necessary as it isolates each test in its own instance of the test class which means that they all
have their own instance of the `InMemorySink`. MSTest however has a different approach and there you may want to use this feature as follows:

```csharp
[TestClass]
public class WhenDemonstratingDisposableFeature
{
    private Logger _logger;

    [TestInitialize]
    public void Initialize()
    {
        _logger?.Dispose();

        _logger = new LoggerConfiguration()
            .WriteTo.InMemory()
            .CreateLogger();
    }

    [TestMethod]
    public void GivenAFoo_BarIsBlah()
    {
        _logger.Information("Foo");

        InMemorySink.Instance
            .Should()
            .HaveMessage("Foo");
    }

    [TestMethod]
    public void GivenABar_BazIsQuux()
    {
        _logger.Information("Bar");

        InMemorySink.Instance
            .Should()
            .HaveMessage("Bar");
    }
}
```

this approach ensures that the `GivenABar_BazIsQuux` does not see any messages logged in a previous test.

## Creating a logger

Loggers are created using a LoggerConfiguration object.
A default initiation would be as follows:

```csharp
var logger = new LoggerConfiguration()
    .WriteTo.InMemory()
    .CreateLogger();
```

### Using an explicit sink instance

By default `WriteTo.InMemory()` uses `InMemorySink.Instance`. When you want to isolate a specific sink instance, pass it explicitly:

```csharp
var sink = new InMemorySink();
var logger = new LoggerConfiguration()
    .WriteTo.InMemory(sink)
    .CreateLogger();
```

### Snapshots

`Snapshot()` creates a read-only copy of the current events so later writes do not affect the assertion target.
You can also filter while taking the snapshot:

```csharp
var errorsOnly = sink.Snapshot(logEvent => logEvent.Level >= LogEventLevel.Error);
```

### Debugger experience

When inspecting `InMemorySink` in a debugger, a debugger proxy presents the captured events as an easy-to-read list with rendered message, template, level, properties, exception, and the original `LogEvent`.

### Minimum level

In this example only Information level logs and higher will be written to the InMemorySink.

```csharp
var logger = new LoggerConfiguration()
    .WriteTo.InMemory(restrictedToMinimumLevel: Events.LogEventLevel.Information)
    .CreateLogger();

```

**Default Level** - if no MinimumLevel is specified, then Verbose level events and [higher](https://github.com/serilog/serilog/wiki/Configuration-Basics#minimum-level) will be processed.

### Dynamic levels

If an app needs dynamic level switching, the first step is to create an instance of LoggingLevelSwitch when the logger is being configured:

```csharp
var levelSwitch = new LoggingLevelSwitch();
```

This object defaults the current minimum level to Information, so to make logging more restricted, set its minimum level up-front:

```csharp
levelSwitch.MinimumLevel = LogEventLevel.Warning;
```

When configuring the logger, provide the switch using MinimumLevel.ControlledBy():

```csharp
var log = new LoggerConfiguration()
    .MinimumLevel.ControlledBy(levelSwitch)
    .WriteTo.InMemory()
    .CreateLogger();
```

Now, events written to the logger will be filtered according to the switch’s MinimumLevel property.

To turn the level up or down at runtime, perhaps in response to a command sent over the network, change the property:

```csharp
levelSwitch.MinimumLevel = LogEventLevel.Verbose;
log.Verbose("This will now be logged");
```
