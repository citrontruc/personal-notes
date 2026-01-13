# Using files

## Table of Content

- [Using files](#using-files)
  - [Table of Content](#table-of-content)
  - [Methods](#methods)
  - [Using JSons](#using-jsons)

## Methods

We have a library **System.IO.Directory** that can do basic operations:

- CreateDirectory
- Exist
- Delete

Using System.IO.File, we can:

- Move
- Copy
- Exist
- Delete
- ReadAllText // Returns an array of text.
- WriteAllText

## Using JSons

You can open json files and deserialize the content to access it as an object. However, you must have defined beforehand your classes and they must have the { get; init; } or { get; set; } properties.

```cs
public static class MenuDataLoader
{
    /// <summary>
    /// Tries to deserialize a Json with information about a Menu.
    /// </summary>
    /// <param name="dataDirectory"></param>
    /// <returns>A Deserialized Json data if the Deserialization operation succeeded. Else, it returns null.</returns>
    /// <exception cref="FileNotFoundException"> Throws an error if the firectory specified does not exist</exception>
    public static JsonData? LoadMenuData(string dataDirectory)
    {
        if (!Directory.Exists(dataDirectory))
        {
            throw new FileNotFoundException($"Could not find the following file: {dataDirectory}");
        }
        string jsonString = File.ReadAllText(dataDirectory);
        var options = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
        JsonData? data = JsonSerializer.Deserialize<JsonData>(jsonString, options);

        return data;
    }
}

public record class JsonData
{
    public List<AssetData>? Assets { get; init; }
    public List<TextData>? Text { get; init; }
}
```
