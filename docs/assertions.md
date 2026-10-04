# Assertions reference

Every assertion in `DragoAnt.Serilog.Sinks.InMemory.Assertions`, with examples. The same code works with FluentAssertions 5-8, AwesomeAssertions 8-9 and Shouldly 4: the package picks the adapter for whichever framework your test project references. Back to the [README](../README.md).

## Example

Let's say you have a class with method implementing some complicated business logic:

```csharp
public class ComplicatedBusinessLogic
{
    private readonly ILogger _logger;

    public ComplicatedBusinessLogic(ILogger logger)
    {
        _logger = logger;
    }

    public string FirstTenCharacters(string input)
    {
        return input.Substring(0, 10);
    }
}
```

A request came in to log a message with the number of characters in the input. So to test that you can create a mock of `ILogger` and assert the method to log was called, however mock setups quickly become very messy (true: this is my opinion!) and assertions on mocks have the same problem when you start verifying values of arguments.

So instead let's use Serilog and a dedicated sink for testing:

```csharp
public class WhenExecutingBusinessLogic
{
    public void GivenInputOfFiveCharacters_MessageIsLogged()
    {
        var logger = new LoggerConfiguration()
            .WriteTo.InMemory()
            .CreateLogger();

        var logic = new ComplicatedBusinessLogic(logger);

        logic.FirstTenCharacters("12345");

        // Use the static Instance property to access the in-memory sink
        InMemorySink.Instance
            .Should()
            .HaveMessage("Input is {count} characters long");
    }
}
```

The test will now fail with `Expected a message to be logged with template \"Input is {count} characters long\" but didn't find any`

Now change the implementation to:

```csharp
public string FirstTenCharacters(string input)
{
    _logger.Information("Input is {count} characters long", input.Length);

    return input.Substring(0, 10);
}
```

Run the test again and it now passes. But how do we ensure this message is only logged once?

To do that, create a new test like so:

```csharp
public void GivenInputOfFiveCharacters_MessageIsLoggedOnce()
{
    /* omitted for brevity */

    InMemorySink.Instance
        .Should()
        .HaveMessage("Input is {count} characters long")
        .Appearing().Once();
}
```

To verify if a message is logged multiple times use `Appearing().Times(int numberOfTimes)`

So now you'll want to verify that the property `count` has the expected value. This builds upon the previous test:

```csharp
public void GivenInputOfFiveCharacters_CountPropertyValueIsFive()
{
    /* omitted for brevity */

    InMemorySink.Instance
        .Should()
        .HaveMessage("Input is {count} characters long")
        .Appearing().Once()
        .WithProperty("count")
        .WithValue(5);
}
```

### Asserting a message appears more than once

Let's say you have a log message in a loop and you want to verify that:

```csharp
public void GivenLoopWithFiveItems_MessageIsLoggedFiveTimes()
{
    /* omitted for brevity */

    InMemorySink.Instance
        .Should()
        .HaveMessage("Input is {count} characters long")
        .Appearing().Times(5);
}
```

### Asserting a message has a certain level

Apart from a message being logged, you'll also want to verify it is of the right level. You can do that using the `WithLevel()` assertion:

```csharp
public void GivenLoopWithFiveItems_MessageIsLoggedFiveTimes()
{
    /* omitted for brevity */

    InMemorySink.Instance
        .Should()
        .HaveMessage("Input is {count} characters long")
        .Appearing().Once()
        .WithLevel(LogEventLevel.Information);
}
```

This also works for multiple messages:

```csharp
public void GivenLoopWithFiveItems_MessageIsLoggedFiveTimes()
{
    logger.Warning("Test message");
    logger.Warning("Test message");
    logger.Warning("Test message");

    InMemorySink.Instance
        .Should()
        .HaveMessage("Test message")
        .Appearing().Times(3)
        .WithLevel(LogEventLevel.Information);
}
```

This will fail with a message: `Expected instances of log message "Test message" to have level Information, but found 3 with level Warning`

### Asserting messages with a pattern

Instead of matching on the exact message you can also match on a certain pattern using the `Containing()` assertion:

```csharp
InMemorySink.Instance
   .Should()
   .HaveMessage()
   .Containing("some pattern")
   .Appearing().Once();
```

which matches on log messages:

- `this is some pattern`
- `some pattern in a message`
- `this is some pattern in a message`

### Asserting messages with a predicate

When matching by template or substring is not enough, you can assert using an arbitrary `Func<LogEvent, bool>` predicate:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage(
        logEvent => logEvent.MessageTemplate.Text.Contains("404"),
        "message containing '404'")
    .Appearing().Once();
```

The inverse is also available through `NotHaveMessage(predicate, description)`.

### Asserting messages have been logged at all (or not!)

When you want to assert that a message has been logged but don't care about what message you can do that with `HaveMessage` and `Appearing`:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage()
    .Appearing().Times(3); // Expect three messages to be logged
```

and of course the inverse is also possible when expecting no messages to be logged:

```csharp
InMemorySink.Instance
    .Should()
    .NotHaveMessage();
```

or that a specific message is not be logged

```csharp
InMemorySink.Instance
    .Should()
    .NotHaveMessage("a specific message");
```

### Asserting properties on messages

When you want to assert that a message has a property you can do that using the `WithProperty` assertion:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage("Message with {Property}")
    .Appearing().Once()
    .WithProperty("Property");
```

To then assert that it has a certain value you would use `WithValue`:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage("Message with {Property}")
    .Appearing().Once()
    .WithProperty("Property")
    .WithValue("property value");
```

Asserting that a message has multiple properties can be accomplished using the `And` constraint:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage("Message with {Property1} and {Property2}")
    .Appearing().Once()
    .WithProperty("Property1")
    .WithValue("value 1")
    .And
    .WithProperty("Property2")
    .WithValue("value 2");
```

When you have a log message that appears a number of times and you want to assert that the value of the log property has the expected values you can do that using the `WithValues` assertion:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage("Message with {Property1} and {Property2}")
    .Appearing().Times(3)
    .WithProperty("Property1")
    .WithValue("value 1", "value 2", "value 3")
```

> **Note:** `WithValue` takes an array of values.

Sometimes you might want to use assertions like `BeLessThanOrEqual()` or `HaveLength()` and in those cases `WithValue` is not very helpful.
Instead you can use `WhichValue<T>()`  to access the value of the log property:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage()
    .Appearing().Once()
    .WithProperty("PropertyOne")
    .WhichValue<string>()
    .Should()
    .HaveLength(3);
```

If the type of the value of the log property does not match the generic type parameter the `WhichValue<T>` method will throw an exception.

> **Note:** This only works for scalar values. When you pass an object as the property value when logging a message Serilog converts that into a string.

### Asserting a property with a destructured object

If you use [object destructuring](https://github.com/serilog/serilog/wiki/Structured-Data#preserving-object-structure):

```csharp
var someObject = new { Foo = "bar", Baz = "quux" };
logger.Information("Hello {@SomeObject}", someObject);
```

and want to assert on properties of the _destructured object_ you can use the `HavingADestructuredObject()` assertion like so:

```csharp
InMemorySink.Instance
    .Should()
    .HaveMessage("Hello {@SomeObject}")
    .Appearing().Once()
    .WithProperty("SomeObject")
    .HavingADestructuredObject()
    .WithProperty("Foo")
    .WithValue("bar");
```

When the property `SomeObject` doesn't hold a destructured object the assertion will fail with the message: `"Expected message "Hello {NotDestructured}" to have a property "NotDestructured" that holds a destructured object but found a scalar value"`

### Building custom assertion extensions

All assertion abstraction interfaces expose a `Subject` property. In addition, top-level assertion types implement `IAssertionsExtension`, so extension authors can call `ToAssertion()` and reuse framework-aware failure handling.

#### Framework-independent extension

```csharp
using System;
using System.Linq;
using DragoAnt.Assertions;
using Serilog.Sinks.InMemory.Assertions;

public static class CustomLogEventAssertions
{
    public static LogEventsAssertions HaveAtLeast(
        this LogEventsAssertions assertions,
        int count,
        string because = "",
        params object[] becauseArgs)
    {
        var extension = assertions.ToAssertion();

        extension.Assert(
            assertions.Subject.Count >= count,
            new FailMessage(
                "Expected at least {0} matching log events, but found {1}.",
                count,
                assertions.Subject.Count),
            because,
            becauseArgs);

        return assertions;
    }
}
```

#### Idempotent extension pattern

Keep custom assertions read-only and deterministic:

- do not mutate `Subject`
- compute result from current state only
- return the same assertion object for chaining

```csharp
using System;
using System.Linq;
using DragoAnt.Assertions;
using Serilog.Sinks.InMemory.Assertions;

public static class CustomLogEventAssertions
{
    public static LogEventsAssertions HaveUniqueMessageTemplates(
        this LogEventsAssertions assertions,
        string because = "",
        params object[] becauseArgs)
    {
        var templates = assertions.Subject
            .Select(e => e.MessageTemplate.Text)
            .ToArray();

        var uniqueCount = templates
            .Distinct(StringComparer.Ordinal)
            .Count();

        assertions.ToAssertion().Assert(
            uniqueCount == templates.Length,
            new FailMessage(
                "Expected matching log events to have unique templates, but found {0} duplicates.",
                templates.Length - uniqueCount),
            because,
            becauseArgs);

        return assertions;
    }
}
```

These extensions are framework-idempotent: the same implementation and failure message shape are used no matter whether the runtime framework is FluentAssertions, AwesomeAssertions, or Shouldly.
