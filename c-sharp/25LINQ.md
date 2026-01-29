# LINQ

## Table of content

- [LINQ](#linq)
  - [Table of content](#table-of-content)
  - [Presentation](#presentation)
  - [Creating Linq methods](#creating-linq-methods)

## Presentation

Query language used to do operation on data inside IEnumerables.

```cs
var numberAsString = numbers.Select(number => number.ToString()).ToList();
var evenNumber = numbers.Select(number => number % 2 == 0).ToList();
var sum = nummbers.Sum();
```

Careful, Linq methods are lazy. Nothing happens until they are forced to run (example: with a ToArray() or a ToList()). Careful if you need a lot of time to access value.

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
