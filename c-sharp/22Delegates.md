# Delegates

## Table of content

- [Delegates](#delegates)
  - [Table of content](#table-of-content)
  - [What is it?](#what-is-it)
  - [Lambda function](#lambda-function)
  - [Actions \& Funcs](#actions--funcs)
  - [Multicasting](#multicasting)
  - [EventHandling](#eventhandling)

## What is it?

A Delegate is a type that safely encapsulates a method. We can have something like the following:

```cs
// We create the delegate type Callback that can be a method that returns void and takes a string as an argument.
public delegate void Callback(string message);

// Create a method for a delegate.
public static void DelegateMethod(string message)
{
    Console.WriteLine(message);
}

// Instantiate the delegate.
Callback handler = DelegateMethod;

// Call the delegate.
handler("Hello World");
```

In dotnet, we rarely use the delegate keyword and rather use the Action and Func.

## Lambda function

A lambda function is an anonymous function most time short. Define it with the => operator.

## Actions & Funcs

Actions don't return results. Both can be defined in a lambda way. Last argument in func is the return type.

```cs
Func<int, int> square = x => x * x; // First argument is tyoe of input, the second is return type.
Func<int, int, bool> testForEquality = (x, y) => x == y;
Func<(int, int, int), (int, int, int)> doubleThem = ns => (2 * ns.Item1, 2 * ns.Item2, 2 * ns.Item3); // You can get creative with elements to put as input.
Func<(int n1, int n2, int n3), (int, int, int)> doubleThem = ns => (2 * ns.n1, 2 * ns.n2, 2 * ns.n3);
```

Lambda functions can be used in LINQ to do operations on lists and arrays.

```cs
int[] numbers = { 2, 3, 4, 5 };
var squaredNumbers = numbers.Select(x => x * x);
Console.WriteLine(string.Join(" ", squaredNumbers));
// Output:
// 4 9 16 25
```

If you have a lot of parameters, you can use the params parameter to pass an array or a collection of parameters.

```cs
var sum = (params IEnumerable<int> values) =>
{
    int sum = 0;
    foreach (var value in values) 
        sum += value;
    
    return sum;
};

var empty = sum();
Console.WriteLine(empty); // 0

var sequence = new[] { 1, 2, 3, 4, 5 };
var total = sum(sequence);
Console.WriteLine(total); // 15
```

Actions can be used for example to debug:

```cs
Action<string> greet = name =>
{
    string greeting = $"Hello {name}!";
    Console.WriteLine(greeting);
};
greet("World");
```

**NOTE** : il est possible d'utiliser var pour définir les types des entrées et sorties :

```cs
var parse = (string s) => int.Parse(s);
```

Il est important par contre que le type de sortie soit fixe ou object.

## Multicasting

We can chain delegates. A reference that is passed to the multicast is passed to each of the methods successively. If there is an exception, we stop dead in our track.

```cs
var obj = new MethodClass();
Callback d1 = obj.Method1;
Callback d2 = obj.Method2;
Callback d3 = DelegateMethod;

//Both types of assignment are valid.
Callback allMethodsDelegate = d1 + d2;
allMethodsDelegate += d3;

//remove Method1
allMethodsDelegate -= d1;

// copy AllMethodsDelegate while removing d2
Callback oneMethodDelegate = (allMethodsDelegate - d2)!;
```

Delegates can be used a lot in Event handling.

## EventHandling

When we have a set of methods that are associated to an Event, we are using delegates.

We have EventArgs to encapsulate data. With mouse events, we can have the position of the mouse or these kind of things.

What do you need? First of, a class to define your EventArgs who contains your arguments. You then need an EventHandler that has a method to **Inwoke** an event passing the EventArgs. In order to define additional elements to pass, define your delegate with public delegate void EventHandler\<TeventArgs>(arguments). We then need to have senders who subscribe to our eventHandler.

```cs
public class NaiveButton<T>
{
    public event EventHandler<ButtonClickedEventArgs>? Clicked;

    // 1. Define the custom Event Arguments
    public class ButtonClickedEventArgs : EventArgs
    {
        public T Value { get; }

        // Event Args
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

public class AsyncVoidExample
{
    static readonly TaskCompletionSource<bool> s_tcs = new TaskCompletionSource<bool>();

    public static async Task MultipleEventHandlersAsync()
    {
        Task<bool> secondHandlerFinished = s_tcs.Task;

        // Sender
        var button = new NaiveButton<bool>();

        // These are the delegates.
        button.Clicked += OnButtonClicked1;
        button.Clicked += OnButtonClicked2Async;
        button.Clicked += OnButtonClicked3;

        Console.WriteLine("Before button.Click() is called...");
        button.Click(true);
        Console.WriteLine("After button.Click() is called...");

        await secondHandlerFinished;
    }

    private static void OnButtonClicked1(object? sender, NaiveButton<bool>.ButtonClickedEventArgs e)
    {
        Console.WriteLine("   Handler 1 is starting...");
        Console.WriteLine(e.Value);
        Task.Delay(100).Wait();
        Console.WriteLine("   Handler 1 is done.");
    }

    private static async void OnButtonClicked2Async(object? sender, NaiveButton<bool>.ButtonClickedEventArgs e)
    {
        Console.WriteLine("   Handler 2 is starting...");
        Task.Delay(100).Wait();
        Console.WriteLine("   Handler 2 is about to go async...");
        await Task.Delay(500);
        Console.WriteLine(e.Value);
        Console.WriteLine("   Handler 2 is done.");
        s_tcs.SetResult(true);
    }

    private static void OnButtonClicked3(object? sender, NaiveButton<bool>.ButtonClickedEventArgs e)
    {
        Console.WriteLine("   Handler 3 is starting...");
        Task.Delay(100).Wait();
        Console.WriteLine(e.Value);
        Console.WriteLine("   Handler 3 is done.");
    }
}
```
