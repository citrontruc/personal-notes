# Misc

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
