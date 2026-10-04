# DragoAnt.Assertions

Detects the assertion framework a test project uses — FluentAssertions 5-8, AwesomeAssertions 8-9 or Shouldly 4 — and loads its adapter, so one assertion helper fails through whichever framework is present.

## Getting started

```sh
dotnet add package DragoAnt.Assertions
dotnet add package AwesomeAssertions
```

Swap `AwesomeAssertions` for `FluentAssertions` or `Shouldly`; the same code keeps working. When AwesomeAssertions and FluentAssertions could both match, AwesomeAssertions wins.

## Usage

```csharp
using DragoAnt.Assertions;

var framework = AssertionUtils.CreateAssertionsFactory().AssertionFramework;

framework.Assert(total > 0, new FailMessage("Expected a positive total, but found {0}.", total));
```

## Documentation

- Building custom, framework-agnostic assertions: https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/docs/assertions.md#building-custom-assertion-extensions
- README: https://github.com/DragoAnt/SerilogSinksInMemory

## Feedback

Issues: https://github.com/DragoAnt/SerilogSinksInMemory/issues
