# DragoAnt.Serilog.Sinks.InMemory

An in-memory Serilog sink for tests: write logs to an `InMemorySink`, then inspect or assert on the captured `LogEvent`s.

## Getting started

```sh
dotnet add package DragoAnt.Serilog.Sinks.InMemory
dotnet add package DragoAnt.Serilog.Sinks.InMemory.Assertions
```

The second package adds fluent assertions (`sink.Should()…`) for FluentAssertions, AwesomeAssertions or Shouldly.

## Usage

```csharp
using Serilog;
using Serilog.Events;
using Serilog.Sinks.InMemory;

var sink = new InMemorySink();
var logger = new LoggerConfiguration()
    .WriteTo.InMemory(sink)
    .CreateLogger();

logger.Warning("Payment {PaymentId} retried", 7);

var warnings = sink.Snapshot(e => e.Level == LogEventLevel.Warning);
```

`WriteTo.InMemory()` without an argument writes to the shared `InMemorySink.Instance`. Namespaces and assembly names are the same as the upstream `Serilog.Sinks.InMemory` package.

## Documentation

- Sink options (snapshots, clearing between tests, levels): https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/sink.md
- README and assertion examples: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
