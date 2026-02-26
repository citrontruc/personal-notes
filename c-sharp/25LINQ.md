# LINQ

## Table of content

- [LINQ](#linq)
  - [Table of content](#table-of-content)
  - [Presentation](#presentation)
  - [Cool method use case](#cool-method-use-case)
    - [Chunk](#chunk)
    - [Zip](#zip)
    - [OrderBy, MinBy, MaxBy](#orderby-minby-maxby)
    - [Takes slices](#takes-slices)
    - [CountBy](#countby)
    - [Index](#index)
    - [Groupby](#groupby)
  - [Creating Linq methods](#creating-linq-methods)
  - [PLINQ](#plinq)

## Presentation

Query language used to do operation on data inside IEnumerables.

```cs
var numberAsString = numbers.Select(number => number.ToString()).ToList();
var evenNumber = numbers.Select(number => number % 2 == 0).ToList();
var sum = numbers.Sum();
```

Careful, Linq methods are lazy. Nothing happens until they are forced to run (example: with a ToArray() or a ToList()). Careful if you need a lot of time to access value.

## Cool method use case

### Chunk

There exists a Chunk method to split an IQueryable into chunks. You can use it to split ueries or if you want to do async operations on parts of our IQueryable.

```cs
var fruits = new List<string> { "Banana", "Pear", "Apple", "Orange" };
var chunks = fruits.Chunk(2); // Size of chunks

foreach (var chunk in chunks)
{
    Console.WriteLine($"[{string.Join(", ", chunk)}]");
}
```

### Zip

Same as python but this time it's:

```cs
var fruits = new List<string> { "Banana", "Pear", "Apple" };
var colors = new List<string> { "Yellow", "Green", "Red" };
var origins = new List<string> { "Ecuador", "France", "Canada" };

var zipped = fruits.Zip(colors, origins);

Console.WriteLine("Now in .NET 6 (Zip with three sequences):");
foreach (var (fruit, color, origin) in zipped)
{
    Console.WriteLine($"{fruit} is {color} and comes from {origin}");
}
```

### OrderBy, MinBy, MaxBy

Does exactly what it says.

```cs
var raceCars = new List<RaceCar>
{
    new RaceCar("Mach 5", 220),
    new RaceCar("Bullet", 180),
    new RaceCar("RoadRunner", 250)
};

// Finding max or min meant sorting or using Aggregate
var fastestCar = raceCars
    .OrderByDescending(car => car.Speed)
    .First();
    
var slowestCar = raceCars
    .OrderBy(car => car.Speed)
    .First();

Console.WriteLine($"Fastest Car: {fastestCar.Name}, Speed: {fastestCar.Speed}");
Console.WriteLine($"Slowest Car: {slowestCar.Name}, Speed: {slowestCar.Speed}");

// ---

var raceCars = new List<RaceCar>
{
    new RaceCar("Mach 5", 220),
    new RaceCar("Bullet", 180),
    new RaceCar("RoadRunner", 250)
};

var fastestCar = raceCars.MaxBy(car => car.Speed);
var slowestCar = raceCars.MinBy(car => car.Speed);

Console.WriteLine($"Fastest Car: {fastestCar.Name}, Speed: {fastestCar.Speed}");
Console.WriteLine($"Slowest Car: {slowestCar.Name}, Speed: {slowestCar.Speed}");
```

### Takes slices

```cs
var fruits = new List<string> { "Banana", "Pear", "Apple", "Orange", "Plum" };

var slice = fruits.Take(2..4);

foreach (var fruit in slice)
{
    Console.WriteLine(fruit);
}
```

### CountBy

Count values on a column.

```cs
var orders = new List<Order>
{
    new Order("Phone", "Electronics", 2, 299.99),
    new Order("Phone", "Electronics", 2, 299.99),
    new Order("TV", "Electronics", 1, 499.99),
    new Order("TV", "Electronics", 1, 499.99),
    new Order("TV", "Electronics", 1, 499.99),
    new Order("Bread", "Groceries", 5, 2.49),
    new Order("Milk", "Groceries", 2, 1.99)
};

var countByName = orders.CountBy(p => p.Name);

foreach (var item in countByName)
{
    Console.WriteLine($"Name: {item.Key}, Count: {item.Value}");
}
```

### Index

Get the value and an index.

```cs
var orders = new List<Order>
{
    new Order("Phone", "Electronics", 2, 299.99),
    new Order("TV", "Electronics", 1, 499.99)
};

foreach (var (index, item) in orders.Index())
{
    Console.WriteLine($"Order #{index}: {item}");
}
```

### Groupby

Method to group items of a list on a key value. Example right under:

```cs
class Pet
{
    public string Name { get; set; }
    public double Age { get; set; }
}

public static void GroupByEx4()
{
    // Create a list of pets.
    List<Pet> petsList =
        new List<Pet>{ new Pet { Name="Barley", Age=8.3 },
                       new Pet { Name="Boots", Age=4.9 },
                       new Pet { Name="Whiskers", Age=1.5 },
                       new Pet { Name="Daisy", Age=4.3 } };

    // Group Pet.Age values by the Math.Floor of the age.
    // Then project an anonymous type from each group
    // that consists of the key, the count of the group's
    // elements, and the minimum and maximum age in the group.
    var query = petsList.GroupBy(
        pet => Math.Floor(pet.Age),
        pet => pet.Age,
        (baseAge, ages) => new
        {
            Key = baseAge,
            Count = ages.Count(),
            Min = ages.Min(),
            Max = ages.Max()
        });

    // Iterate over each anonymous type.
    foreach (var result in query)
    {
        Console.WriteLine("\nAge group: " + result.Key);
        Console.WriteLine("Number of pets in this age group: " + result.Count);
        Console.WriteLine("Minimum age: " + result.Min);
        Console.WriteLine("Maximum age: " + result.Max);
    }

    /*  This code produces the following output:

        Age group: 8
        Number of pets in this age group: 1
        Minimum age: 8.3
        Maximum age: 8.3

        Age group: 4
        Number of pets in this age group: 2
        Minimum age: 4.3
        Maximum age: 4.9

        Age group: 1
        Number of pets in this age group: 1
        Minimum age: 1.5
        Maximum age: 1.5
    */
}
```

The example above transforms the data but you don't need to do that to use the method, here is an example that just takes the values and puts them in a list:

```cs
public Dictionary<string, List<Trade>> RegroupTradesByTraderIdWithGroupby()
{
    Dictionary<string, List<Trade>> result = new();
    var inter = _dailyTrades.GroupBy(trade => trade.TraderId, (TraderId, trades) => new
            {
                Key = TraderId,
                AllTrades = trades
            }
        );

    foreach (var value in inter)
    {
        result[value.Key] = value.AllTrades.ToList();
    }

    return result;
}
```

## Creating Linq methods

You just need a static extension method that operates on IEnumerables.

```cs
public static class MyLinq
{
    public static IEnumerable<T> LinqMethod<T>( this IEnumerable<T> source, Func<T, T> selector)
    {
        foreach (T item in source)
        {
            Console.WriteLine($"Applying transformation to {item}");
            yield return selector(item);
        }
    }
}
```

A practical linq method is the .Aggregate method that lets you do a math operation on the whole list to return an aggregated version.

```cs
int productOfAllElementsOnList = myList.Aggregate(1, (product, next) => product * next);

int[] ints = { 4, 8, 8, 3, 9, 0, 7, 8, 2 };

// Count the even numbers in the array, using a seed value of 0.
int numEven = ints.Aggregate(0, (total, next) =>
                                    next % 2 == 0 ? total + 1 : total);
```

## PLINQ

For parallel LINQ operations. We want to improve performance of operations.
Careful, operations need to be independant.

```cs
var source = new [] {1, 2, 3, 4};
var query = source.AsParallel().Select(Compute); // Compute query in parallel
var result = query.Sum();
```

AsParallel has a bunch of options available (with cancellation, with degress of parallelism...)

Don't abuse of AsParallel, because it adds overhead because you need to analyze the query.

Trouble with parallel operations is that you don't always know in which order operations will take place. It is possible to do AsParallel().AsOrdered() in order to make surre that the order is kept. Be careful with what you do.
