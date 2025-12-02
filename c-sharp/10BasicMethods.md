# Some basic methods to make your life easier

## Create a list with the same element repeating

```cs
List<bool> listFindMissingElement = Enumerable.Repeat(false, N + 1).ToList();
```

## SubString

We have substring with the first element to tell us at which letter we start and the second to ask us how many letters we take.

```cs
str  = "GeeksForGeeks"
str.Substring(5,3);
```

## String.IndexOf

Obtenir l'index d'un élément. On peut indiquer le startIndex à partir duquel regarder.


## ^3

Dans une liste, prendre les éléments à partir de la fin.

## array[2..5]

Obtenir une trnache d'un array.

## Searches in lists

```cs
var result = people
    .Where(p => p.Age > 18)
    .OrderBy(p => p.LastName)
    .Select(p => p.FirstName);
```
