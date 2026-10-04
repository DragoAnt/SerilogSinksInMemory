# Differences from upstream

This repository is a fork of [serilog-contrib/SerilogSinksInMemory](https://github.com/serilog-contrib/SerilogSinksInMemory) (earlier hosted at [sandermvanvliet/SerilogSinksInMemory](https://github.com/sandermvanvliet/SerilogSinksInMemory)). Compared with upstream tag `2.0.0.0`, it differs in these user-visible ways. Back to the [README](../README.md).

- NuGet package IDs are `DragoAnt.Serilog.Sinks.InMemory` and `DragoAnt.Serilog.Sinks.InMemory.Assertions`. Namespaces and assembly names remain `Serilog.Sinks.InMemory*`.
- Assertion framework discovery and adapter loading are now encapsulated in standalone packages `DragoAnt.Assertions` and `DragoAnt.Assertions.Abstractions`.
- Packages now target `netstandard2.0` instead of `netstandard2.1`, widening compatibility for older test projects.
- `WriteTo.InMemory(outputTemplate: ...)` is no longer part of the public API. Use `WriteTo.InMemory()` for the default singleton sink, or `WriteTo.InMemory(sink, ...)` to write into an explicit `InMemorySink` instance.
- The sink and assertions APIs now support predicate-based filtering via `InMemorySink.Snapshot(Func<LogEvent, bool>)`, `HaveMessage(Func<LogEvent, bool>, ...)`, and `NotHaveMessage(Func<LogEvent, bool>, ...)`.
- `InMemorySink` now uses a debugger proxy so watch windows show a friendlier view of each log event, including rendered message, template, level, properties, exception, and the original `LogEvent`.
- Assertion abstractions now expose `Subject`, and top-level assertion types provide `ToAssertion()` helpers for building custom assertion extensions in a framework-agnostic way.
- NuGet consumption of the assertions package is more reliable: packaged assertion adapters are exposed transitively, framework detection probes `AppContext.BaseDirectory`, and `AwesomeAssertions` is preferred when both it and `FluentAssertions` could otherwise match.
