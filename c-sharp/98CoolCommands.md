# Cool commands

## Table of content

- [Cool commands](#cool-commands)
  - [Table of content](#table-of-content)
  - [Lookup in dictionary](#lookup-in-dictionary)
  - [Initialization with anonymous types](#initialization-with-anonymous-types)
  - [with](#with)

## Lookup in dictionary

```cs
var configLookup = new Dictionary<int, (int Min, int Max)>()
{
    [2] = (4, 10),
    [4] = (10, 20),
    [6] = (0, 23)
};

if (configLookup.TryGetValue(4, out (int Min, int Max) range))
{
    Console.WriteLine($"Found range: min is {range.Min}, max is {range.Max}");
}
// Output: Found range: min is 10, max is 20
```

## Initialization with anonymous types

```cs
var title = "Software Engineer";
var department = "Engineering";
var salary = 75000;

// Using projection initializers.
var employee = new { title, department, salary };

// Equivalent to explicit syntax:
// var employee = new { title = title, department = department, salary = salary };

Console.WriteLine($"Title: {employee.title}, Department: {employee.department}, Salary: {employee.salary}");
```

Option: **Cascade initializations**

```cs
var product = new Product();
var bonus = new { note = "You won!" };
var shipment = new { address = "Nowhere St.", product };
var shipmentWithBonus = new { address = "Somewhere St.", product, bonus };

// End result should be { address = Somewhere St., product = { product = stuff }, bonus = { note = You won! } }
```

Option: **Multiple initializations**

```cs
var anonArray = new[] { new { name = "apple", diam = 4 }, new { name = "grape", diam = 1 }};
```

## with

Lets you change a value but won't let you add one. You can only do non destructive mutations.

```cs
var apple = new { Item = "apples", Price = 1.35 };
var onSale = apple with { Price = 0.79 };;
```
