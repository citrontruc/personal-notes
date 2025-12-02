# Errors

## Throw

Basic errors can be raised using throw new Error.

Possible de créer tout type d'erreurs en faisant hériter d'autres erreurs. On peut throw des erreurs dans le cas d'expressions :

```cs
string first = args.Length >= 1 
    ? args[0]
    : throw new ArgumentException("Please supply at least one argument.");
```

## try-catch

Afin d'essayer d'éviter les erreurs, on peut les intercepter avec try-catch. Exemple ci-dessous :

```cs
try
{
    ProcessShapes(shapeAmount);
}
catch (Exception e)
{
    LogError(e, "Shape processing failed.");
    throw;
}
```

Il vaut mieux préciser le type d'erreur que l'on souhaite attraper. Pour cela, on peut définir autant de cas de figure que l'on souhaite et autant de catch que l'on souhaite. On peut même introduire des when quand on veut appliquer des filtres sur les valeurs d'erreurs que l'on désire conserver. On peut vraiment en faire beaucoup.

```cs
try
{
    var result = await ProcessAsync(-3, 4, cancellationToken);
    Console.WriteLine($"Processing succeeded: {result}");
}
catch (Exception e) when (e is ArgumentException || e is DivideByZeroException)
{
    Console.WriteLine($"Processing failed: {e.Message}");
}
catch (ArgumentException e)
{
    Console.WriteLine($"Processing failed: {e.Message}");
}
catch (OperationCanceledException)
{
    Console.WriteLine("Processing is cancelled.");
}
```
