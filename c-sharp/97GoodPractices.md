# Practices

## Table of Content

- [Practices](#practices)
  - [Table of Content](#table-of-content)
  - [POO](#poo)
  - [Booleans](#booleans)
  - [using](#using)
  - [StringComparison](#stringcomparison)

## POO

Favor Object composition over class inheritance. If we have inheritance, we see the "insides" of a class. If the parent class changes, we don't always see the consequences for our child class. In composition, you know that the object you are using must obey an interface but you don't need more.

## Booleans

Prefix your booleans:

```cs
// active
bool isActive;
// access
bool hasAccess;
// continue
bool shouldContinue;
// edit
bool canEdit;
```

## using

Used for imports. Keep it outside of the namespace to make it cleaner and also avoid inconsistent behaviours (with global using and other).

Your scope should be the file.

## StringComparison

Do not use ToLower(), it creates a new string and allocates memory, use **StringComparison.OrdinalIgnoreCase**. It compares the binary value.

```cs
string str1 = "hello";
string str2 = "HELLO";

// Returns true
bool isEqual = string.Equals(str1, str2, StringComparison.OrdinalIgnoreCase);

// Dictionary usage
var dict = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
```

Another example using the Distinct method in dotnet. Here, we treat capitalized and normal values in the same way:

```cs
result.Distinct(StringComparer.Ordinal)
```
