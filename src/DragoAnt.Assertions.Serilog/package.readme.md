# DragoAnt.Assertions.Serilog

The Serilog log-event adapters behind `DragoAnt.Serilog.Sinks.InMemory.Assertions`: one adapter per supported framework (FluentAssertions 5-8, AwesomeAssertions 8-9, Shouldly 4), selected at run time.

## Getting started

You normally get this package as a dependency of the in-memory sink assertions:

```sh
dotnet add package DragoAnt.Serilog.Sinks.InMemory.Assertions
```

## Usage

Assert through the sink assertions package; see the README for examples:

```csharp
sink.Should().HaveMessage("Order {OrderId} placed").Appearing().Once();
```

## Documentation

- All assertions: https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/assertions.md
- README: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
