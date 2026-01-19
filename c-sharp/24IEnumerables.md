# IEnumerables

## Table of Content

- [IEnumerables](#ienumerables)
  - [Table of Content](#table-of-content)
  - [Presentation](#presentation)
  - [Limits](#limits)
  - [LINQ](#linq)
  - [Yield statement](#yield-statement)
  - [Iterators](#iterators)
  - [Warning](#warning)

## Presentation

You can only iterate on IEnumerables one element at a time. They are reference type so if you modify a value it will be modified for everybody.

## Limits

IEnumerables do not have a Length, you can't access the element on position I, you can only iterate on an IEnumerable.

You can however use foreach on an IEnumerable.

## LINQ

As long as the element implements IEnumerable, you can use Linq on it.

## Yield statement

Use yield statement to return elements as an enumerator.
It has nothing to do with asynchronicity (for asynchronicity, use **await**).

Example:

```cs
Console.WriteLine(string.Join(" ", TakeWhilePositive(new int[] {2, 3, 4, 5, -1, 3, 4})));
// Output: 2 3 4 5

Console.WriteLine(string.Join(" ", TakeWhilePositive(new int[] {9, 8, 7})));
// Output: 9 8 7

IEnumerable<int> TakeWhilePositive(IEnumerable<int> numbers)
{
    foreach (int n in numbers)
    {
        if (n > 0)
        {
            yield return n;
        }
        else
        {
            yield break;
        }
    }
}
```

You can do things in an asynchronous way with yield await.

The result object being an enumerator lets you run the foreach (int value in result){ DoStuff(); }. Here is an example right under:

```cs
// 1. Call the generator method.
// This does NOT execute the loop yet; it just creates the sequence object (IEnumerable<int>).
IEnumerable<int> evenNumberSequence = GenerateEvenNumbers(startRange, endRange);

// 2. Store the numbers in a List<int> using ToList().
// Calling ToList() forces the generator to execute completely, fetching and storing every yielded result.
List<int> evenNumbersList = evenNumberSequence.ToList();

// --- Alternative: Manual Storage (without LINQ) ---
/*
List<int> manualList = new List<int>();
// The generator executes step-by-step as the foreach loop requests the next item
foreach (int number in GenerateEvenNumbers(10, 30))
{
    manualList.Add(number);
}
Console.WriteLine("\nManual List (10 to 30): " + string.Join(", ", manualList));
*/
```

## Iterators

An iterator is a method that returns the elements of an IEnumerable.
Using an iterator is good when we want to add logs during the iteration.

```cs
IEnumerable<int> SimpleIterator()
{
    for (var item in dataset)
    {
        yield return item;
    }
}
```

Iterators are lazy. You point to the function but if you had a sleep command inside, it won't trigger. We need to use a foreach to iterate on an iterator.

```cs
IEnumerable<int> SimpleIterator()
{
    yield return 123;
    yield return 568;
}

var yieldResults = SimpleIterator(); // No int returned here. We have a pointer on the function.

// We are running code, we don't call a List ==> It will take time everytime.
// However, we don't hold anything in memory. There is no list.
foreach (var number in yieldResults)
[
    Console.WriteLine(number);
]
```

## Warning

Because of the lazy execution, it can be complicated to use yield and Iterators. If you want to be cautious, do not use them.
