# Misc

## All methods of initializing arrays

```cs
string[] array = new string[2]; // creates array of length 2, default values
string[] array = new string[] { "A", "B" }; // creates populated array of length 2
string[] array = { "A" , "B" }; // creates populated array of length 2
string[] array = new[] { "A", "B" }; // creates populated array of length 2
string[] array = ["A", "B"]; // creates populated array of length 2
```

## Codewars

```cs
// return masked string
  public static string Maskify(string cc)
  {
    if (cc.Length <= 4){
      return cc;
    }
    return new string('#', cc.Length - 4) + cc.Substring(cc.Length - 4);
  }
```

Exercice : dans un supermarché, des clients arrivent aux caisses. ils prennent la première caisse vide. Dans combien de temps est-ce que tout le monde aura fait ses courses.

```cs
using System;
using System.Collections.Generic;
using System.Linq;

public class Kata
{
    public static long QueueTime(int[] customers, int n)
    {
      long totalWaitTime = 0;
      int[] supermarketQueues = new int[n];
      for (int i = 0; i < customers.Length; i++){
        int min = supermarketQueues.Min();
        
        // We compute how long it would take to free a queue.
        if (min > 0){
          supermarketQueues = supermarketQueues.Select(x => x - min).ToArray();
          totalWaitTime += min;
        }
        int indexEmptyQueue = Array.FindIndex(supermarketQueues, num => num == 0);
        supermarketQueues[indexEmptyQueue] = customers[i];
      }
      totalWaitTime += supermarketQueues.Max();
        return totalWaitTime;
    }
}
```

## Books

1. The Pragmatic Programmer (Andy Hunt & Dave Thomas) :thumbsup:

    Practical lessons for every stage of your career.
    It teaches adaptability, curiosity, and continuous learning.

2. The Clean Coder (Robert C. Martin)

    Learn professionalism, discipline, and how to take ownership.
    Being a great developer isn't just about writing code — it's about attitude.

3. Head First Design Patterns (Eric Freeman & Elisabeth Robson)

    A fun and visual way to learn design patterns.
    You'll finally understand how patterns actually work in real projects.

4. Head First Software Architecture (Raju Gandhi, Mark Richards, Neal Ford)

    A friendly, hands-on guide to thinking like an architect.
    Great for developers who want to move into architecture.

5. Building Evolutinary Architectures (Neal Ford, Rebecca Parsons, Patrick Kua)

    Learn how to design systems that can adapt and grow over time.
    Change is constant — this book teaches you how to handle it.

6. Enterprise Architecture Patterns (Martin Fowler)

    Understand how to structure and connect large systems.
    Perfect for enterprise or long-term, large-scale projects.

7. Designing Data-Intensive Applications (Martin Kleppmann)

    A must-read for modern system design.
    It explains databases, scaling, and data processing in clear language.

8. Domain-Driven Design (Eric Evans)

    Learn to model complex business logic with simplicity.
    It will change how you talk to both devs and business people.

9. Building Microservices (Sam Newman)

    A complete guide to creating distributed systems.
    Full of real-world trade-offs and practical lessons.

10. Patterns of Enterprise Application Architecture (Martin Fowler)

    Classic reference for solving common design problems.
    Once you read it, you'll start seeing these patterns everywhere.

Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy

The Mythical Man-Month: Essays on Software Engineering :thumbsup:

Peopleware: Productive Projects and Teams

Dinosaur Brains: Dealing with All Those Impossible People at Work

Fundamentals of Software Architecture: An Engineering Approach O'Reilly

Software Architecture: The Hard Parts: Modern Trade-Off Analyses for Distributed Architectures O'Reilly
