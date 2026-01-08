# Keywords

## Table of content

- [Keywords](#keywords)
  - [Table of content](#table-of-content)
  - [Hashtable vs Dictionary](#hashtable-vs-dictionary)
  - [var](#var)
  - [internal vs private](#internal-vs-private)
  - [init](#init)
  - [sealed](#sealed)
  - [Cool stuff for inheritance](#cool-stuff-for-inheritance)
  - [struct](#struct)
  - [Records](#records)
  - [enum](#enum)
  - [static](#static)
  - [readonly](#readonly)
  - [ref](#ref)
  - [zip](#zip)
  - [Yield statement](#yield-statement)
  - [Partial classes](#partial-classes)
  - [with](#with)

## Hashtable vs Dictionary

HashTable stocke les objets en tant qu'object tandis que les dictionnaires est fortement typé. Hashtable embarque un peu de gestion de la concurrence.

## var

variable type is determined at compilation.

WARNING: there is a **dynamic** keyword that defines the type of the variable at runtime example: dynamic name = "test"; But it is risky.

Var is called anonymous type.

```cs
// Tuple with named elements.
var tupleProduct = (Name: "Widget", Price: 19.99M);
Console.WriteLine($"Tuple: {tupleProduct.Name} costs ${tupleProduct.Price}");

// Equivalent example using anonymous types.
var anonymousProduct = new { Name = "Widget", Price = 19.99M };
Console.WriteLine($"Anonymous: {anonymousProduct.Name} costs ${anonymousProduct.Price}");
```

## internal vs private

private is class scope and internal is assembly scope.

## init

Value can only be set by the constructor.

```cs
string Id { get; init; } = Id;
```

However, if your field is an array or a list, you can change the value it contains. Pas conseillé mais possible.

Can be useful when you have an automatic creation of a field:

```cs
class Person_InitExampleFieldProperty
{
    public int YearOfBirth
    {
        get;
        init
        {
            field = (value <= DateTime.Now.Year)
                ? value
                : throw new ArgumentOutOfRangeException(nameof(value), "Year of birth can't be in the future");
        }
    }
}

class Person_InitExample
{
     private int _yearOfBirth;

     public int YearOfBirth
     {
         get { return _yearOfBirth; }
         init { _yearOfBirth = value; }
     }
}
```

## sealed

You can't inherit from a sealed class. You can't override a sealed attribute.

## Cool stuff for inheritance

When inheriting from a class, in your constructor you can call the previous contructor.
Runs the constructor of the base class before running the other constructor.
Does not work for other method overrides.

```cs
public class DerivedClass : BaseClass
{
    // The derived class constructor
    public DerivedClass(int value) : base(value) // <-- This is the key part
    {
        // Derived class initialization logic runs *after* the base constructor finishes
        Console.WriteLine("DerivedClass constructor running.");
    }
}
```

## struct

When building structs, you can override operators (+, -, ==, !=). Do not forget to also override methods ToString, Equals and GetHashCode.

C sharp **struct** have value semantics. Aka, variable values are copied on assignment. p2 = p1 while create a copy of p1 in p2.

- A simple data container.
- Fields are usually public by default.
- No or limited behavior (methods) depending on the language.
- Often value-type semantics (copied on assignment) in languages like C/C++ and C#.
- Used for lightweight, immutable-ish, or small data aggregates.

## Records

In C# (specifically starting with C# 9), when you use the record keyword by itself, you are implicitly declaring a record class.

- Designed to represent data with meaning (entities, rows, DTOs).
- Built-in support for value equality (two records with same fields are equal).
- Typically immutable by default.
- Often comes with automatically generated features: equals, hash, toString, constructors, etc.
- In languages like C# or Java, records can still have methods and validation but remain primarily data-centric.

Records peuvent être des record class et des record struct. Record struct peut gérer l'expression with qui crée un clone de la valeur. Exemple :

```cs
var p2 = p1 with {Y=30};
```

Un des rares vrais avantages est la facilité pour définir mais ce n'est pas énorme non plus.

<https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record>

```cs
// Record Struct - Concise definition
public record struct Color(int R, int G, int B);

// Traditional Struct - More verbose
public struct ColorStruct 
{
    public int R { get; set; }
    public int G { get; set; }
    public int B { get; set; }
}
```

Isolate data access logic from the rest of the application.

```cs
public interface IProductRepository
{
    Task<Product> GetByIdAsync(int id);
    Task<List<Product>> GetAllAsync();
    Task AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public Task<Product> GetByIdAsync(int id) =>
        _context.Products.FindAsync(id).AsTask();

    public Task<List<Product>> GetAllAsync() =>
        _context.Products.ToListAsync();

    public Task AddAsync(Product product) =>
        _context.Products.AddAsync(product).AsTask();

    public Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
        return _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var p = await GetByIdAsync(id);
        if (p != null)
        {
            _context.Products.Remove(p);
            await _context.SaveChangesAsync();
        }
    }
}
```

Records to represent data. Immutable, can have inheritance. Les records viennent souvent avec des méthodes equals, hash et toString.

## enum

In csharp, **enum** is a value type defined by a set of named constants.

```cs
enum Season {
    Spring,
    Summer,
    Autumn,
    Winter
}
```

**p++ and ++p** (pre increment and post increment). Both increment, the first one returns the value pre-incrementation, the second one post incrementation.

## static

**static** keywords mean values does not belong to instances of the class but to the class itself. static class then all the elements must be static.

## readonly

Mots clés : readonly peut être assigné dans constructeur puis nulle part ailleurs, const est fixé lors de la compilation.
exemple :

```cs
public class Test
{
    const int c = 10;
    readonly int test = 10;
    public Test(int value)
    {
        test = value;
    }
}
```

## ref

Mot clé ref dans argument d'une méthode quand on veut passer la référence d'une variable. A ce moment, l'objet est modifié. ref n'exige pas que l'objet soit initialisé. out semble pareil mais besoin que l'objet soit initialisé.

## zip

```cs
var diffs = values
    .Zip(values.Skip(1), (current, next) => next - current)
    .ToList();
```

translation: attach to values the value values.Skip(1) and for x, y, do operation.

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

You can do thinks in an asynchronous way with yield await.

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

## Partial classes

Classes that are defined in multiple places in order to add information:

```cs
// This is in Employee_Part1.cs
public partial class Employee
{
    public void DoWork()
    {
        Console.WriteLine("Employee is working.");
    }
}

// This is in Employee_Part2.cs
public partial class Employee
{
    public void GoToLunch()
    {
        Console.WriteLine("Employee is at lunch.");
    }
}

//Main program demonstrating the Employee class usage
public class Program
{
    public static void Main()
    {
        Employee emp = new Employee();
        emp.DoWork();
        emp.GoToLunch();
    }
}

// Expected Output:
// Employee is working.
// Employee is at lunch.
```

## with

Créer une copie d'un objet avec des éléments différents.

```cs
var shipment = new { address = "Nowhere St.", product = "stuff" };
var testWith = shipment with {address = "Autre address"};
```
