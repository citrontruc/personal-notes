# Practices

## Table of Content

- [Practices](#practices)
  - [Table of Content](#table-of-content)
  - [Booleans](#booleans)
  - [using](#using)

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
