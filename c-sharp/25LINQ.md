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
  - [Creating Linq methods](#creating-linq-methods)

## Presentation

Query language used to do operation on data inside IEnumerables.

```cs
var numberAsString = numbers.Select(number => number.ToString()).ToList();
var evenNumber = numbers.Select(number => number % 2 == 0).ToList();
var sum = nummbers.Sum();
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
