# Generics

Principal avantage est la réutilisation et le fait qu'on n'a pas besoin de faire du boxing ou du typecasting.

## Principe

Avoir un placeholder/template que l'on remplace au moment opportun par un type donné. Le type de la variable est défini par le programme qui utilise notre objet avec un generics.

```cs
public class NaiveButton<T>
{
    public event EventHandler<ButtonClickedEventArgs>? Clicked;

    // 1. Define the custom Event Arguments
    public class ButtonClickedEventArgs : EventArgs
    {
        public T Value { get; }

        public ButtonClickedEventArgs(T value)
        {
            this.Value = value;
        }
    }

    public void Click(T value)
    {
        Console.WriteLine("Somebody has clicked a button. Let's raise the event...");
        ButtonClickedEventArgs buttonClickedEventArgs = new(value);
        Clicked?.Invoke(this, buttonClickedEventArgs);
        Console.WriteLine("All listeners are notified.");
    }
}
```

Note : on peut utiliser des Generics en tant que Generics et en temps qu'arguments d'une fonction :

```cs
public class ClsCalculator
{
    public static bool AreEqual<T>(T value1, T value2)
    {
        return value1.Equals(value2);
    }
}
```

Il est aussi possible de faire des méthodes qui prennent plusieurs generics en arguments :

```cs
public void GenericMethod<T1, T2>(T1 Param1, T2 Param2)
{
    Console.WriteLine($"Parameter T1 type: {typeof(T1)}: Parameter T2 type: {typeof(T2)}");
    Console.WriteLine($"Parameter 1: {Param1} : Parameter 2: {Param2}");
}
```

## Ajout de contraintes

Il est possible de rajouter des contraintes sur T en précisant where T : ...

```cs
where T : EntityBase, IAggregateRoot, new()
```

## Covariance and Contravariance

https://stackoverflow.com/questions/2662369/covariance-and-contravariance-real-world-example

Lié aux Generics. Fonctionne uniquement pour les generics.

**Covariance (out T)**
You can use a more derived type where a less derived type is expected.
Exemple : robot de cuisine prend des aliments pour préparer un repas. On peut mettre une banane qui est un élément plus dérivé.

**Contravariance (in T)**
You can use a less derived type where a more derived type is expected.
Un promeneur pour chien peut prendre un mammifère pour le promener.

Exemple manifestation en code :

```cs
// Source - https://stackoverflow.com/a
// Posted by CSharper, modified by community. See post 'Timeline' for change history
// Retrieved 2025-12-01, License - CC BY-SA 3.0

public interface ICovariant<out T> { }
public interface IContravariant<in T> { }

public class Covariant<T> : ICovariant<T> { }
public class Contravariant<T> : IContravariant<T> { }

public class Fruit { }
public class Apple : Fruit { }

public class TheInsAndOuts
{
    public void Covariance()
    {
        ICovariant<Fruit> fruit = new Covariant<Fruit>();
        ICovariant<Apple> apple = new Covariant<Apple>();

        Covariant(fruit);
        Covariant(apple); //apple is being upcasted to fruit, without the out keyword this will not compile
    }

    public void Contravariance()
    {
        IContravariant<Fruit> fruit = new Contravariant<Fruit>();
        IContravariant<Apple> apple = new Contravariant<Apple>();

        Contravariant(fruit); //fruit is being downcasted to apple, without the in keyword this will not compile
        Contravariant(apple);
    }

    public void Covariant(ICovariant<Fruit> fruit) { }

    public void Contravariant(IContravariant<Apple> apple) { }
}
```
