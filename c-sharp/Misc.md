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

Exerice : dans un supermarché, des clients arrivent aux caisses. ils prennent la première caisse vide. Dans combien de temps est-ce que tout le monde aura fait ses courses.

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
