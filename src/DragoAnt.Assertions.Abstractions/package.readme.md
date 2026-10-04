# DragoAnt.Assertions.Abstractions

Framework-agnostic contracts for writing assertions once and running them on FluentAssertions, AwesomeAssertions or Shouldly: `AssertionFramework`, `FailMessage`, `IAssertionsExtension` and the `ToAssertion()` helper.

## Getting started

Most projects get this package through `DragoAnt.Assertions` or `DragoAnt.Serilog.Sinks.InMemory.Assertions`. Reference it directly when a library only needs the contracts:

```sh
dotnet add package DragoAnt.Assertions.Abstractions
```

## Usage

An assertion type that implements `IAssertionsExtension` can fail through the active framework:

```csharp
using DragoAnt.Assertions;

public static class CountAssertions
{
    public static void HaveAtLeast(this IAssertionsExtension assertions, int actual, int expected)
        => assertions.ToAssertion().Assert(
            actual >= expected,
            new FailMessage("Expected at least {0} items, but found {1}.", expected, actual));
}
```

## Documentation

- Custom assertion extensions: https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/assertions.md#building-custom-assertion-extensions
- README: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
