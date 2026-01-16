# Guard Clauses

## Table of Content

- [Guard Clauses](#guard-clauses)
  - [Table of Content](#table-of-content)
  - [Fail fast](#fail-fast)
  - [Libraries](#libraries)
    - [Ardalis](#ardalis)
    - [Guard clauses toolkit](#guard-clauses-toolkit)

## Fail fast

If something is wrong, you should fail fast in order to detect the error.

```cs
public void ProcessOrder(Order order)
{
    if (order is null)
    {
        throw new ArgumentNullException();
    }
    if (order.Quantity <= 0)
    {
        throw new ArgumentOutOfRangeException();
    }
    if (order.ProductId is null)
    {
        throw new ArgumentException();
    }

    // Continue processing...
}

// Most of the "classic" configuration evaluations have a dedicated evaluation method

public void ProcessOrder(Order order)
{
    ArgumentNullException.ThrowIfNull(order);
    ArgumentOutOfRangeException.ThrowIfNegativeOrZero(order.Quantity);
    ArgumentException.ThrowIfNullOrWhiteSpace(order.ProductId);

    // Continue processing...
}

ArgumentOutOfRangeException.ThrowIfEqual<int>(value, 42, nameof(value));
ArgumentOutOfRangeException.ThrowIfGreaterThan<int>(amount, max, nameof(amount));
ArgumentOutOfRangeException.ThrowIfLessThan<decimal>(price, 0, nameof(price));
ArgumentOutOfRangeException.ThrowIfNegative<int>(count, nameof(count));
ArgumentOutOfRangeException.ThrowIfZero<double>(factor, nameof(factor));
```

## Libraries

There is a library called Ardalis which lets you define a lot of guard clauses. Another example of library is the guard clauses toolkit by microsoft.

### Ardalis

```sh
Install-Package Ardalis.GuardClauses
```

```cs
Guard.Against.Null // (throws if input is null)
Guard.Against.NullOrEmpty // (throws if string, guid or array input is null or empty)
Guard.Against.NullOrWhiteSpace // (throws if string input is null, empty or whitespace)
Guard.Against.OutOfRange // (throws if integer/DateTime/enum input is outside a provided range)
Guard.Against.EnumOutOfRange // (throws if an enum value is outside a provided Enum range)
Guard.Against.OutOfSQLDateRange // (throws if DateTime input is outside the valid range of SQL Server DateTime values)
Guard.Against.Zero // (throws if number input is zero)
Guard.Against.Expression // (use any expression you define)
Guard.Against.InvalidFormat // (define allowed format with a regular expression or func)
Guard.Against.NotFound // (similar to Null but for use with an id/key lookup; throws a NotFoundException)

// Using the same namespace will make sure your code picks up your
// extensions no matter where they are in your codebase.
namespace Ardalis.GuardClauses
{
    public static class FooGuard
    {
        public static void Foo(this IGuardClause guardClause,
            string input, 
            [CallerArgumentExpression("input")] string? parameterName = null)
        {
            if (input?.ToLower() == "foo")
                throw new ArgumentException("Should not have been foo!", parameterName);
        }
    }
}
```

### Guard clauses toolkit

```sh
Install-Package Microsoft.Toolkit.Diagnostics.Guard
```

```cs
Guard
    .Against.Null(order)
    .Against.OutOfRange(order.Quantity, 1, 100)
    .Against.NullOrWhiteSpace(order.ProductId);
```
