# Async return types

## Table of content

- [Async return types](#async-return-types)
  - [Table of content](#table-of-content)
  - [Task](#task)
  - [EventHandling](#eventhandling)
  - [Errors to avoid](#errors-to-avoid)
    - [Blocking](#blocking)
    - [Async void](#async-void)
    - [Errors not awaited](#errors-not-awaited)
    - [.Result and .Wait()](#result-and-wait)
  - [AsyncStreams](#asyncstreams)
  - [Tasks](#tasks)
    - [Creating tasks](#creating-tasks)
    - [Cancelling tasks](#cancelling-tasks)
  - [Waiting on multiple tasks](#waiting-on-multiple-tasks)
  - [ConfigureAwait](#configureawait)

## Task

By default, a task is the result of an async method that returns no values. Task\<TResult> if you return a value.

void for EventHandling.

Async and await relieve the server from work. The server is free to do another task while the await happens.

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

If we have to call a function that we have to await multiple times, it can be more optimal to launch all your processes together and wait for them all to finish rather than to await them one after another.

```cs
// Don't:

task1 = await WaitAndApologizeAsync();
task2 = await WaitAndApologizeAsync();

// Do

task1 = WaitAndApologizeAsync();
task2 = WaitAndApologizeAsync();
Task.WaitAll([task1, task2]);
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

### Blocking

**WARNING**: Avoid blocking. NEVER use GetDataAsync().Wait() or GetDataAsync().Result, because it will block your program.

### Async void

Errors in an async void cannot be caught. Indeed, the error does not return an error task but makes the thread crash.

EventHandlers are the only elements to use async void.

### Errors not awaited

If you have a task that is not awaited and it fails, the task is a failure and nothing happens because it is forgotten.

### .Result and .Wait()

These methods make your code synchronous again because you wait for something in a synchronous way.

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

## Tasks

### Creating tasks

Basic idea: We create a task, we define a very iportant operation and then do Task.Run(SomeMethod);

```cs
// Compiler infers the type of return so we don't need to specify the value returned.
Task<T> task = Task.Run(() => { return new T();});
Task task = Task.Run(() => { });

// We can have a chain of tasks to run.
task.ContinueWith(OtherOperation, TaskContinuationOptions.NotOnFaulted);
// Use this to catch exceptions.
task.ContinueWith(OtherOperation, TaskContinuationOptions.OnlyOnFaulted);
// ContinueWith only reference the previous task. if you have an error on the first task, the second won't run with NotOnFaulted but the third one would ==> Make sure tasks only run if the previous task completed.
task.ContinueWith((completedTask) => 
// We can call result because the previous task will have already completed before we run this code.
    { var resultOfPreviousOperation = completedTask.Result;} 
    );
```

Careful: we can't update the UI thread by default when we use another thread. Only the UI thread can modify the UI. Use the Dispatcher to communicate with the UI thread, it pulls work back to the UI thread.

### Cancelling tasks

If you want to cancel a task, you need a cancellation token. We pass a token and the thread checks if a cancellation was requested. In order to do so, first create a cancellation token source and if you want to cancel the tasks attached to thhe cancellation token source, use the .Cancel() method.

There is a method called .CancelAfter() that can be used in order to set a timeout value.

Next taks and task continuation will not run if task was cancelled. However, the current task will continue ==> You need to handle the cancellation gracefully (for example with a break).

**NOTE**: HttpClient handles this way more easily.

## Waiting on multiple tasks

Regroup tasks in list and launch Task.WhenAll. Returns an array of all the results.

```cs
var task1 = Task.Run(() => { return "string1"; });
var task2 = Task.Run(() => { return "string2"; });

var tasks = new [] {task1, task2};

string[] result = await Task.WhenAll(tasks);
```

We can add a timeout by creating a delay task and then looking at which one finishes first from our delay task and our other task.

```cs
Task task1 = Task.Run( );
Task task2 = Task.Delay(2000);

var completedTask = await Task.WhenAny(task1, task2);
if (completedTask == task2)
{
    //Cancel all our tasks using our cancellationTokenSource.
}
```

## ConfigureAwait

Specify that we allow the task to run on a different thread, we don't need the current thread to be available. Mostly unused now. Can create problems if you have an UI thread. Use it if you build a library.
