# New Versions

## Table of Content

- [New Versions](#new-versions)
  - [Table of Content](#table-of-content)
  - [C-sharp 10](#c-sharp-10)
    - [File Based App](#file-based-app)
    - [Namespaces](#namespaces)
    - [Implicit Usings](#implicit-usings)
    - [Global Usings](#global-usings)
    - [Struct](#struct)
    - [record struct](#record-struct)
    - [Server Sent events](#server-sent-events)
    - [Extension keyword](#extension-keyword)

## C-sharp 10

### File Based App

You can now launch a prgm from only a .cs file. You don't need a csproj and an sln.

### Namespaces

FileScoped Namespace. You can define namespaces for the whole file with ; instead of having to encapsuulate the whole file with {}.

```cs
// Can only use one per file.
namespace Shared.Dtos;
public record DtoBase(int Id);

public record EventPrice(int Id);
```

it is the same thing as:

```cs
// Can still be used.
namespace Shared.Dtos{
    public record DtoBase(int Id);

    public record EventPrice(int Id);
}
```

### Implicit Usings

By enabling implicit usings in your project file, you don't need to type using for common namespaces like system. The list of implicit usings depends on the sdk that we are using. We can also change it with ItemGroup in the project file.

### Global Usings

Have a GlobalUsing.cs file where you put your global usings files.

### Struct

In C# 10, you can have a default constructor for structs that takes no arguments. Using the readonly keyword, you can make your structs immutable.

Example:

```cs
Coordinate c = new Coordinate {X = 12, Y = 42};
// Creates copy with changed value.
Coordinate cChange = c with { X = 13 };

public readonly struct Coordinate{
    public Coordinate(){
        X = 0;
        Y = 0;
    }
    public int X {get; init;}
    public int Y {get; init;}
}
```

### record struct

record structs are new in C# 10. They are immutable and come with a default constructor.

```cs
Coordinate c = new Coordinate(12, 42);
var (x, y) = c;

public readonly record struct Coordinate(int X, int Y) { }
```

You can also evaluate == with record structs.

### Server Sent events

```cs
public record StockPriceEvent(string Id, string Symbol, decimal Price, DateTime Timestamp);

public class StockService
{
    public async IAsyncEnumerable<StockPriceEvent> GenerateStockPrices(
       [EnumeratorCancellation] CancellationToken cancellationToken)
    {
       var symbols = new[] { "MSFT", "AAPL", "GOOG", "AMZN" };

       while (!cancellationToken.IsCancellationRequested)
       {
          var symbol = symbols[Random.Shared.Next(symbols.Length)];
          var price = Math.Round((decimal)(100 + Random.Shared.NextDouble() * 50), 2);
          var id = DateTime.UtcNow.ToString("o");

          yield return new StockPriceEvent(id, symbol, price, DateTime.UtcNow);

          await Task.Delay(TimeSpan.FromSeconds(2), cancellationToken);
       }
    }
}
```

### Extension keyword

See keyword file. Idea is that you can define multiple extensions in the same group.
