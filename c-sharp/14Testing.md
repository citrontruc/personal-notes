# Testing

## Table of content

- [Testing](#testing)
  - [Table of content](#table-of-content)
  - [Tests](#tests)
  - [Best practices](#best-practices)
  - [Conventions](#conventions)
    - [Naming](#naming)
    - [Content](#content)
    - [Simplicity](#simplicity)
    - [All the elements used by the test are present in the test](#all-the-elements-used-by-the-test-are-present-in-the-test)
    - [You don't need to test private methods](#you-dont-need-to-test-private-methods)
  - [XUnit](#xunit)

## Tests

Using MSTest. WARNING: mstest is an aging framework, it is recommended to use XUnit instead.

```bash
dotnet new mstest
dotnet sln add <path to project>
dotnet add reference <path to project>
```

```bash
dotnet new mstest --framework net9.0 -n MyTestProject
```

Warning! Sometimes, everything is fine but MSTest will still mark it as an error in vscode.

In path to project, use /

<https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-mstest>

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

## XUnit

XUnit has two main test methods: Fact and Theory

**Fact**: Represents a test method that should be true under the given circumstances. It is typically used for tests that have no inputs or outputs. Use it only if you g=have results that don't change.

```cs
public class MathOperations
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

public class MathOperationsTests
{
    [Fact]
    public void Add_ReturnsCorrectSum()
    {
        // Arrange
        MathOperations math = new MathOperations();

        // Act
        int result = math.Add(3, 5);

        // Assert
        Assert.Equal(8, result);
    }
}
```

**Theory**: Represents a parametrized test method that receives arguments from data sources. The theory is tested with different sets of data to ensure that it behaves correctly under various conditions.

```cs
public class MathOperations
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

public class MathOperationsTests
{
    [Theory]
    [InlineData(3, 5, 8)]   // First parameter: 3, Second parameter: 5, Expected result: 8
    [InlineData(2, 2, 4)]   // First parameter: 2, Second parameter: 2, Expected result: 4
    [InlineData(10, 5, 15)] // First parameter: 10, Second parameter: 5, Expected result: 15
    public void Add_ReturnsCorrectSum(int a, int b, int expected)
    {
        // Arrange
        MathOperations math = new MathOperations();

        // Act
        int result = math.Add(a, b);

        // Assert
        Assert.Equal(expected, result);
    }
}
```

**MemberData** is an attribute in xUnit.net used to specify a method that will provide the test data for a parameterized test. The method specified by MemberData must return IEnumerable<object[]>, where each object array represents a set of test data.

```cs
using System.Collections.Generic;
using Xunit;

public class MathOperations
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

public class MathOperationsTests
{
    // Method providing test data
    public static IEnumerable<object[]> TestData()
    {
        yield return new object[] { 3, 5, 8 };
        yield return new object[] { 2, 2, 4 };
        yield return new object[] { 10, 5, 15 };
    }

    [Theory]
    [MemberData(nameof(TestData))]
    public void Add_ReturnsCorrectSum(int a, int b, int expected)
    {
        // Arrange
        MathOperations math = new MathOperations();

        // Act
        int result = math.Add(a, b);

        // Assert
        Assert.Equal(expected, result);
    }
}
```

**TestDataClass**: same thing as previous but this time it is a class that provides the test data. ClassData specifies a class that provides the test data for a parameterized test. The class specified by ClassData must implement IEnumerable<object[]>, where each object array represents a set of test data.

```cs
using System.Collections.Generic;
using Xunit;

public class MathOperations
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

public class MathOperationsTests
{
    // Class providing test data
    public class TestDataClass : IEnumerable<object[]>
    {
        public IEnumerator<object[]> GetEnumerator()
        {
            yield return new object[] { 3, 5, 8 };
            yield return new object[] { 2, 2, 4 };
            yield return new object[] { 10, 5, 15 };
        }

        IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
    }

    [Theory]
    [ClassData(typeof(TestDataClass))]
    public void Add_ReturnsCorrectSum_ClassData(int a, int b, int expected)
    {
        // Arrange
        MathOperations math = new MathOperations();

        // Act
        int result = math.Add(a, b);

        // Assert
        Assert.Equal(expected, result);
    }
}
```
