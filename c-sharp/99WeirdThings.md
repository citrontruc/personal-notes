# Weird Stuff

## Table of content

- [Weird Stuff](#weird-stuff)
  - [Table of content](#table-of-content)
  - [String and StringBuilders](#string-and-stringbuilders)
  - [Boxed and unboxed variables](#boxed-and-unboxed-variables)
  - [Return Types](#return-types)

## String and StringBuilders

String is immutable. Every modification creates a new instance. This costs memory and time when you do many concatenations.

StringBuilder is mutable. It updates the internal buffer without creating new objects. It’s faster and more efficient for repeated appends, inserts, or replacements.

## Boxed and unboxed variables

You can "box" a variable in an object. You can then unbox it later to access its value.

Boxing happens because value types are stored differently from reference types. When you assign a value type to something typed as object (or any interface), the runtime allocates a new object on the heap and copies the value into it.

The boxed value is a reference-type wrapper created at runtime. It isn’t the same value you had on the stack.

Example: you have a list of objects so you can track in the same list elements of different nature that are all objects.

```cs
var list = new ArrayList();

// Boxing happens here (int → object)
list.Add(42);
list.Add(DateTime.Now); // also boxed

foreach (object item in list)
{
    // Unboxing required when retrieving
    if (item is int number)
    {
        Console.WriteLine(number);
    }
}
```

## Return Types

I have a class to create PagedLists (lists with additional properties). However, my retrun type is IEnumerable. My PagedList is therefore converted to an IEnumerable ==> It loses its additional properties. If I wanted to have them, I should either define an IResponse type and add the properties at the time of respones or change my Response so that it is not an IEnumerable.

```cs
public async Task<IEnumerable<Album>> GetAllAlbums(
    MusicDbContext db,
    int? pageSize,
    int? pageNumber
)
{
    (int correctPageSize, int correctPageNumber) =
        AlbumParameters.CorrectPaginationParameters(
            _defaultAlbumParameters,
            pageSize,
            pageNumber
        );

    return await PagedList<Album>.ToPagedListAsync(
        db.Albums.AsNoTracking().OrderBy(a => a.ArtistName).ThenBy(a => a.Name),
        correctPageSize,
        correctPageNumber
    );
}
```
