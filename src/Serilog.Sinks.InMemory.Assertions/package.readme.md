# DragoAnt.Serilog.Sinks.InMemory.Assertions

Fluent assertions for the Serilog in-memory sink that work the same on FluentAssertions 5-8, AwesomeAssertions 8-9 and Shouldly 4.

## Getting started

```sh
dotnet add package DragoAnt.Serilog.Sinks.InMemory
dotnet add package DragoAnt.Serilog.Sinks.InMemory.Assertions
```

Reference one of the supported assertion frameworks in the test project; the package detects it at run time and reports failures through it.

## Usage

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Sinks.InMemory;
using Serilog.Sinks.InMemory.Assertions;

var sink = new InMemorySink();
var logger = new LoggerConfiguration()
    .WriteTo.InMemory(sink)
    .CreateLogger();

logger.Information("Order {OrderId} placed", 42);

sink.Should()
    .HaveMessage("Order {OrderId} placed")
    .Appearing().Once()
    .WithLevel(LogEventLevel.Information)
    .WithProperty("OrderId")
    .WithValue(42);
```

Also available: `Appearing().Times(n)`, `Containing(...)`, predicate matching, `NotHaveMessage(...)`, `WhichValue<T>()` and `HavingADestructuredObject()`.

## Documentation

- All assertions with examples: https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/assertions.md
- README: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
