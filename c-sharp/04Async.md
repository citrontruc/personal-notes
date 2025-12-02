# Async return types

## Task

By default, a task is the result of an async method that returns no values. Task<TResult> if you return a value.

void for EventHandling.

Basic example of an async method.
```cs
public static async Task DisplayCurrentInfoAsync()
{
    await WaitAndApologizeAsync();

    Console.WriteLine($"Today is {DateTime.Now:D}");
    Console.WriteLine($"The current time is {DateTime.Now.TimeOfDay:t}");
    Console.WriteLine("The current temperature is 76 degrees.");
}

static async Task WaitAndApologizeAsync()
{
    await Task.Delay(2000);

    Console.WriteLine("Sorry for the delay...\n");
}
```

Basic Example with a return value:
```cs
var getLeisureHoursTask = GetLeisureHoursAsync();

string message =
    $"Today is {DateTime.Today:D}\n" +
    "Today's hours of leisure: " +
    $"{await getLeisureHoursTask}";

Console.WriteLine(message);

public static async Task ShowTodaysInfoAsync()
{
    string message =
        $"Today is {DateTime.Today:D}\n" +
        "Today's hours of leisure: " +
        $"{await GetLeisureHoursAsync()}";

    Console.WriteLine(message);
}

static async Task<int> GetLeisureHoursAsync()
{
    DayOfWeek today = await Task.FromResult(DateTime.Now.DayOfWeek);

    int leisureHours =
        today is DayOfWeek.Saturday || today is DayOfWeek.Sunday
        ? 16 : 5;

    return leisureHours;
}
```

Waiting on multiple calls

```cs
var t1 = GetDataAsync();
var t2 = GetDataAsync();
var results = await Task.WhenAll(t1, t2);
```

## EventHandling

Example of a button that throws a bool to all the events that subscribed to it.

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

public class AsyncVoidExample
{
    static readonly TaskCompletionSource<bool> s_tcs = new TaskCompletionSource<bool>();

    public static async Task MultipleEventHandlersAsync()
    {
        Task<bool> secondHandlerFinished = s_tcs.Task;

        var button = new NaiveButton<bool>();

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

## Errors to avoid

**WARNING**: Avoid blocking. NEVER use GetDataAsync().Wait() or GetDataAsync().Result, because it will block your program.

## AsyncStreams

Get data as they arrive in an IAsyncEnumerable

```cs
async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}

await foreach (var n in GetNumbersAsync())
{
    Console.WriteLine(n);
}
```
