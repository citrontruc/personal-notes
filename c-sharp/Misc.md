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
