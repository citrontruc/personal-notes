# Testing

## Table of content

- [Testing](#testing)
  - [Table of content](#table-of-content)
  - [Best practices](#best-practices)
  - [Conventions](#conventions)
    - [Naming](#naming)
    - [Content](#content)
    - [Simplicity](#simplicity)
    - [All the elements used by the test are present in the test](#all-the-elements-used-by-the-test-are-present-in-the-test)
    - [You don't need to test private methods](#you-dont-need-to-test-private-methods)

## Best practices

Tests must be repeatable, evaluate how the system works, detect bugs, guide you towards improving the system and avoid regression.

When you test components, you might have dependencies. In order to make tests possible, we have to Mock these dependencies. That is where **interfaces** come in play! Since interfaces are contracts, you can create mocks based on these interfaces.

## Conventions

### Naming

The name of your test should consist of three parts:

- Name of the method being tested
- Scenario under which the method is being tested
- Expected behavior when the scenario is invoked

### Content

The "Arrange, Act, Assert" pattern is a common approach for writing unit tests. As the name implies, the pattern consists of three main tasks:

- Arrange your objects, create, and configure them as necessary
- Act on an object
- Assert that something is as expected

```cs
[Fact]
public void Add_EmptyString_ReturnsZero()
{
    // Arrange
    var stringCalculator = new StringCalculator();

    // Act
    var actual = stringCalculator.Add("");

    // Assert
    Assert.Equal(0, actual);
}
```

### Simplicity

Don't use coding logic in tests, don't try to do intermediary operations. If you want to do multiple operations, run a test multiple times with one input rather than doing a for loop. In the example below, we run three times the test with three data points.

```cs
[Theory]
[InlineData("0,0,0", 0)]
[InlineData("0,1,2", 3)]
[InlineData("1,2,3", 6)]
public void Add_MultipleNumbers_ReturnsSumOfNumbers(string input, int expected)
{
    var stringCalculator = new StringCalculator();

    var actual = stringCalculator.Add(input);

    Assert.Equal(expected, actual);
}
```

### All the elements used by the test are present in the test

Don't try to reuse elements between tests or have class values. Create in the test all the elements needed by the test. For example, have a method to create standardized elements to run your tests.

```cs
private StringCalculator CreateDefaultStringCalculator()
{
    return new StringCalculator();
}
```

### You don't need to test private methods

Private methods are the details of the implementation. They are not meant to be called from the exterior. Normally, you wouldn't need to test them. Test the public methods that use them.
