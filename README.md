# DragoAnt.Serilog.Sinks.InMemory

An in-memory Serilog sink for tests, with fluent log assertions that work the same on FluentAssertions, AwesomeAssertions and Shouldly.

[![Build](https://img.shields.io/github/actions/workflow/status/DragoAnt/SerilogSinksInMemory/dotnet.yml?branch=main)](https://github.com/DragoAnt/SerilogSinksInMemory/actions/workflows/dotnet.yml)
[![NuGet](https://img.shields.io/nuget/v/DragoAnt.Serilog.Sinks.InMemory)](https://www.nuget.org/packages/DragoAnt.Serilog.Sinks.InMemory)
[![Downloads](https://img.shields.io/nuget/dt/DragoAnt.Serilog.Sinks.InMemory)](https://www.nuget.org/packages/DragoAnt.Serilog.Sinks.InMemory)
[![License](https://img.shields.io/github/license/DragoAnt/SerilogSinksInMemory)](https://github.com/DragoAnt/SerilogSinksInMemory/blob/main/LICENSE)
![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)

## In short

- **What:** write logs to an `InMemorySink` in your tests, then assert on them — template, count, level, properties, destructured objects — instead of verifying calls on a mocked `ILogger`.
- **One assertion API, three frameworks:** the assertions package detects FluentAssertions 5-8, AwesomeAssertions 8-9 or Shouldly 4 in your test project and reports failures through it.
- **A maintained fork** of [serilog-contrib/SerilogSinksInMemory](https://github.com/serilog-contrib/SerilogSinksInMemory), published as `DragoAnt.*` packages; namespaces and assembly names stay `Serilog.Sinks.InMemory*`, so switching is a package-id change. What differs from upstream: [docs/differences-from-upstream.md](./docs/differences-from-upstream.md).

## Packages

| Package | What it is | NuGet |
| --- | --- | --- |
| `DragoAnt.Serilog.Sinks.InMemory` | The sink | [![NuGet](https://img.shields.io/nuget/v/DragoAnt.Serilog.Sinks.InMemory)](https://www.nuget.org/packages/DragoAnt.Serilog.Sinks.InMemory) |
| `DragoAnt.Serilog.Sinks.InMemory.Assertions` | `sink.Should()…` log assertions | [![NuGet](https://img.shields.io/nuget/v/DragoAnt.Serilog.Sinks.InMemory.Assertions)](https://www.nuget.org/packages/DragoAnt.Serilog.Sinks.InMemory.Assertions) |
| `DragoAnt.Assertions` | Detects the assertion framework at run time and loads its adapter | [![NuGet](https://img.shields.io/nuget/v/DragoAnt.Assertions)](https://www.nuget.org/packages/DragoAnt.Assertions) |
| `DragoAnt.Assertions.Abstractions` | Framework-agnostic contracts for writing your own assertions | [![NuGet](https://img.shields.io/nuget/v/DragoAnt.Assertions.Abstractions)](https://www.nuget.org/packages/DragoAnt.Assertions.Abstractions) |
| `DragoAnt.Assertions.Serilog`, `.Serilog.Abstractions` | The Serilog-specific adapters and contracts behind the assertions package | [![NuGet](https://img.shields.io/nuget/v/DragoAnt.Assertions.Serilog)](https://www.nuget.org/packages/DragoAnt.Assertions.Serilog) |

You normally install only the first two; the rest come in as dependencies.

## Install

```sh
dotnet add package DragoAnt.Serilog.Sinks.InMemory
dotnet add package DragoAnt.Serilog.Sinks.InMemory.Assertions
```

## Quick start

```csharp
using Serilog;
using Serilog.Sinks.InMemory;
using Serilog.Sinks.InMemory.Assertions;
using Xunit;

public sealed class CheckoutTests
{
    [Fact]
    public void Logs_the_placed_order()
    {
        var sink = new InMemorySink();
        var logger = new LoggerConfiguration()
            .WriteTo.InMemory(sink)
            .CreateLogger();

        logger.Information("Order {OrderId} placed", 42);

        sink.Should()
            .HaveMessage("Order {OrderId} placed")
            .Appearing().Once()
            .WithProperty("OrderId")
            .WithValue(42);
    }
}
```

A failing assertion is reported through your assertion framework, like any other failed check. `WriteTo.InMemory()` without an argument writes to the shared `InMemorySink.Instance`; pass your own sink, as above, to keep parallel tests apart.

More: [all assertions](./docs/assertions.md) (levels, patterns, predicates, destructured objects, custom extensions) · [sink options](./docs/sink.md) (snapshots, clearing between tests, minimum and dynamic levels).

## Assertions for any framework

`DragoAnt.Assertions` lets you write one assertion helper that fails through whichever framework the test project uses — no branching per framework:

```csharp
using DragoAnt.Assertions;

var framework = AssertionUtils.CreateAssertionsFactory().AssertionFramework;

framework.Assert(total > 0, new FailMessage("Expected a positive total, but found {0}.", total));
```

Log-assertion types expose `Subject` and `ToAssertion()`, so a custom extension such as `HaveAtLeast(3)` is a few lines: see [building custom assertion extensions](./docs/assertions.md#building-custom-assertion-extensions).

## Compatibility

The packages target `netstandard2.0` and depend on Serilog 4.x. The assertions support FluentAssertions 5, 6, 7 and 8, AwesomeAssertions 8 and 9, and Shouldly 4; when both AwesomeAssertions and FluentAssertions could match, AwesomeAssertions wins.

## Changelog and releases

[Changelog.md](./Changelog.md) · maintainers: [RELEASING.md](./RELEASING.md).

## License

[MIT](./LICENSE). Originally created by Sander van Vliet; this fork is maintained by DragoAnt.
