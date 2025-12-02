# Threads

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

Afin d'éviter ce genre de mécanisme, il faut que
- Tous les locks se fassent dans le même ordre.
- Avoir des timeouts et des mécanismes pour réessayer plus tard.
- Limiter la durée des locks et les scopes des locks afin d'éviter des conflits.

Il existe cependant des manières annexes de gérer les choses. Il est notamment conseillé de faire de l'**async**.
