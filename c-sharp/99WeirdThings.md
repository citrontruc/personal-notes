# Weird Stuff

## String and StringBuilders

String is immutable. Every modification creates a new instance. This costs memory and time when you do many concatenations.

StringBuilder is mutable. It updates the internal buffer without creating new objects. It’s faster and more efficient for repeated appends, inserts, or replacements.

## Boxed and unboxed variables

You can "box" a variable in an object. You can then unbox it later to access its value.

Boxing happens because value types are stored differently from reference types. When you assign a value type to something typed as object (or any interface), the runtime allocates a new object on the heap and copies the value into it.

The boxed value is a reference-type wrapper created at runtime. It isn’t the same value you had on the stack.

Example: you have a list of objects so you can track in the same list elements of different nature that are all objects.

```cs
var list = new ArrayList();

// Boxing happens here (int → object)
list.Add(42);
list.Add(DateTime.Now); // also boxed

foreach (object item in list)
{
    // Unboxing required when retrieving
    if (item is int number)
    {
        Console.WriteLine(number);
    }
}
```

