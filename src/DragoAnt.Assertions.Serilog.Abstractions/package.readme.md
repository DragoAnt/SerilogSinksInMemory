# DragoAnt.Assertions.Serilog.Abstractions

Framework-agnostic contracts for Serilog log-event assertions, used by `DragoAnt.Serilog.Sinks.InMemory.Assertions` and its per-framework adapters.

## Getting started

You normally get this package as a dependency:

```sh
dotnet add package DragoAnt.Serilog.Sinks.InMemory.Assertions
```

## Usage

The assertion types (`LogEventsAssertions` and friends) expose `Subject` and `ToAssertion()`, so you can add your own log assertions that work on every supported framework. See the custom-extension example in the documentation.

## Documentation

- Custom assertion extensions: https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/assertions.md#building-custom-assertion-extensions
- README: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
