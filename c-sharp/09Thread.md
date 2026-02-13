# Threads

## Table of content

- [Threads](#threads)
  - [Table of content](#table-of-content)
  - [How does it work?](#how-does-it-work)
  - [locks](#locks)
  - [New version of locks](#new-version-of-locks)
  - [Lock structures](#lock-structures)
  - [Semaphores](#semaphores)
  - [Deadlock](#deadlock)
  - [Thread-safe Structures](#thread-safe-structures)

## How does it work?

Idée est de faire de l'asynchrone. on possède un thread pool. On coupe le code. On lance une première méthode et tout ce qui est après le await est sur le premier thread disponible. Le nombre de thread existant est géré par le langage. C'est dynamique.

Quand on a await, on change de thread constamment. Quand on fait .Wait(), on attend de façon synchrone.

Quand on a des UI, on possède un thread pour l'UI qui possède des droits privilégiés. Le framework sait quand on affiche une UI qu'il doit y avoir un thread UI. WPF est le framework utilisé dans notre cas.

Si on a une UI avec un .Wait() sur une méthode qui contient un await à l'intérieur, on obtient un deadlock car le thread attend quelque chose qui attend et rien ne se passe.

Afin de pouvoir faire en sorte que des frontends laissent d'autres threads se charger des tâches, on peut utiliser .ConfigureAwait(false).

Synchronisation Context. Parfois, des threads sont dans un Synchronisation Context. Toutes les opérations faites sont dans ce contexte qui est un thread pool à part.

Note : les éléments d'UI ne sont pas thread safe car sinon, ce serait trop gourmand. C'est pour ça qu'ils ne le sont pas. En contrepartie, on ne possède qu'un seul thread.

## locks

Le but du lock est de faire en sorte qu'un morceau de code ne soit accédé que par un processus.
Il est possible d'avoir des locks successifs.

```cs
using System;
using System.Threading;

class Program {
    private static int counter = 0;
    private static readonly object lockObject = new object();

    static void IncrementCounter() {
        for (int i = 0; i < 1000; i++) {
            lock (lockObject) {
                counter++;
            }
        }
    }

    static void Main() {
        Thread t1 = new Thread(IncrementCounter);
        Thread t2 = new Thread(IncrementCounter);
        
        t1.Start();
        t2.Start();
        t1.Join();
        t2.Join();

        Console.WriteLine("Final Counter: " + counter);
    }
}
```

## New version of locks

There is a new version of lock that is more optimal. System.Threading.Lock.
Here is an example with a bank account lock system.

This new lock system has multiple methods (EnterScope). The following example only uses basic lock functionalities.

```cs
using System;
using System.Threading.Tasks;

public class Account
{
    // Use `object` in versions earlier than C# 13
    private readonly System.Threading.Lock _balanceLock = new();
    private decimal _balance;

    public Account(decimal initialBalance) => _balance = initialBalance;

    public decimal Debit(decimal amount)
    {
        if (amount < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(amount), "The debit amount cannot be negative.");
        }

        decimal appliedAmount = 0;
        lock (_balanceLock)
        {
            if (_balance >= amount)
            {
                _balance -= amount;
                appliedAmount = amount;
            }
        }
        return appliedAmount;
    }

    public void Credit(decimal amount)
    {
        if (amount < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(amount), "The credit amount cannot be negative.");
        }

        lock (_balanceLock)
        {
            _balance += amount;
        }
    }

    public decimal GetBalance()
    {
        lock (_balanceLock)
        {
            return _balance;
        }
    }
}

class AccountTest
{
    static async Task Main()
    {
        var account = new Account(1000);
        var tasks = new Task[100];
        for (int i = 0; i < tasks.Length; i++)
        {
            tasks[i] = Task.Run(() => Update(account));
        }
        await Task.WhenAll(tasks);
        Console.WriteLine($"Account's balance is {account.GetBalance()}");
        // Output:
        // Account's balance is 2000
    }

    static void Update(Account account)
    {
        decimal[] amounts = [0, 2, -3, 6, -2, -1, 8, -5, 11, -6];
        foreach (var amount in amounts)
        {
            if (amount >= 0)
            {
                account.Credit(amount);
            }
            else
            {
                account.Debit(Math.Abs(amount));
            }
        }
    }
}
```

## Lock structures

There are some built in components for effective lock handling.

Indeed, **lock** blocks all access to a piece of code. **ReaderWriterLockSlim** allows to differentiate read and write operations. The example below shows the difference between these two operations:

```cs
using System;
using System.Collections.Generic;
using System.Threading;

class Program
{
    static ReaderWriterLockSlim rwLock = new ReaderWriterLockSlim();
    static List<int> sharedData = new List<int>();

    static void Main()
    {
        // Start some reader threads
        for (int i = 0; i < 3; i++)
        {
            new Thread(ReadData).Start();
        }

        // Start some writer threads
        for (int i = 0; i < 2; i++)
        {
            new Thread(WriteData).Start();
        }
    }

    static void ReadData()
    {
        while (true)
        {
            rwLock.EnterReadLock();
            try
            {
                Console.WriteLine($"Reading data: {string.Join(",", sharedData)}");
            }
            finally
            {
                rwLock.ExitReadLock();
            }
            Thread.Sleep(500);
        }
    }

    static void WriteData()
    {
        Random rnd = new Random();
        while (true)
        {
            rwLock.EnterWriteLock();
            try
            {
                int value = rnd.Next(100);
                sharedData.Add(value);
                Console.WriteLine($"Wrote data: {value}");
            }
            finally
            {
                rwLock.ExitWriteLock();
            }
            Thread.Sleep(1000);
        }
    }
}
```

## Semaphores

Instead of using locks, you can use semaphores in order to limit the number of threads accessing a piece of code. In the same way as lockSlicm, you now have SemaphoreSlim:

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
    private static SemaphoreSlim semaphore;
    // A padding interval to make the output more orderly.
    private static int padding;

    public static void Main()
    {
        // Create the semaphore.
        semaphore = new SemaphoreSlim(0, 3);
        Console.WriteLine("{0} tasks can enter the semaphore.",
                          semaphore.CurrentCount);
        Task[] tasks = new Task[5];

        // Create and start five numbered tasks.
        for (int i = 0; i <= 4; i++)
        {
            tasks[i] = Task.Run(() =>
            {
                // Each task begins by requesting the semaphore.
                Console.WriteLine("Task {0} begins and waits for the semaphore.",
                                  Task.CurrentId);
                
                int semaphoreCount;
                semaphore.Wait();
                try
                {
                    Interlocked.Add(ref padding, 100);

                    Console.WriteLine("Task {0} enters the semaphore.", Task.CurrentId);

                    // The task just sleeps for 1+ seconds.
                    Thread.Sleep(1000 + padding);
                }
                finally {
                    semaphoreCount = semaphore.Release();
                }
                Console.WriteLine("Task {0} releases the semaphore; previous count: {1}.",
                                  Task.CurrentId, semaphoreCount);
            });
        }

        // Wait for half a second, to allow all the tasks to start and block.
        Thread.Sleep(500);

        // Restore the semaphore count to its maximum value.
        Console.Write("Main thread calls Release(3) --> ");
        semaphore.Release(3);
        Console.WriteLine("{0} tasks can enter the semaphore.",
                          semaphore.CurrentCount);
        // Main thread waits for the tasks to complete.
        Task.WaitAll(tasks);

        Console.WriteLine("Main thread exits.");
    }
}
// The example displays output like the following:
//       0 tasks can enter the semaphore.
//       Task 1 begins and waits for the semaphore.
//       Task 5 begins and waits for the semaphore.
//       Task 2 begins and waits for the semaphore.
//       Task 4 begins and waits for the semaphore.
//       Task 3 begins and waits for the semaphore.
//       Main thread calls Release(3) --> 3 tasks can enter the semaphore.
//       Task 4 enters the semaphore.
//       Task 1 enters the semaphore.
//       Task 3 enters the semaphore.
//       Task 4 releases the semaphore; previous count: 0.
//       Task 2 enters the semaphore.
//       Task 1 releases the semaphore; previous count: 0.
//       Task 3 releases the semaphore; previous count: 0.
//       Task 5 enters the semaphore.
//       Task 2 releases the semaphore; previous count: 1.
//       Task 5 releases the semaphore; previous count: 2.
//       Main thread exits.
```

## Deadlock

Situation où le code est bloqué parce que deux procédures doivent chacun accéder à une ressource que l'autre procédure lock. Exemple ci-dessous :

```cs
using System;
using System.Threading;

class Program {
    private static readonly object lock1 = new object();
    private static readonly object lock2 = new object();

    static void Thread1() {
        lock (lock1) {
            Thread.Sleep(100); // Simulating some work
            lock (lock2) {
                Console.WriteLine("Thread 1 completed.");
            }
        }
    }

    static void Thread2() {
        lock (lock2) {
            Thread.Sleep(100); // Simulating some work
            lock (lock1) {
                Console.WriteLine("Thread 2 completed.");
            }
        }
    }

    static void Main() {
        Thread t1 = new Thread(Thread1);
        Thread t2 = new Thread(Thread2);
        
        t1.Start();
        t2.Start();
        t1.Join();
        t2.Join();
    }
}
```

Afin d'éviter ce genre de mécanisme, il faut que :

- Tous les locks se fassent dans le même ordre.
- Avoir des timeouts et des mécanismes pour réessayer plus tard.
- Limiter la durée des locks et les scopes des locks afin d'éviter des conflits.

## Thread-safe Structures

There are some natively thread-safe structures: example is the ConcurrentDictionary or ConcurrentHashmap. Use the specific methods to access elements.

List\<T> for example is not thread safe. Consider using a ConcurrentBag\<T> instead. You can use this to chain calls and update a value on the screen as tasks get results.
