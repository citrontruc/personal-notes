# LLM

## Table of content

- [LLM](#llm)
  - [Table of content](#table-of-content)
  - [RAG](#rag)

## RAG

In order to do RAG, we need to store data in a data store. In order to do so, C# has an SQLVector type that can be used for embedding:

```cs
public sealed class Document
{
    public Guid Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public SqlVector<float> Embedding { get; set; }
}
```

Basic code to generate an embedding using phi3:mini.

```cs
using Microsoft.Extensions.AI;
using OllamaSharp;

IEmbeddingGenerator<string, Embedding<float>> generator =
    new OllamaApiClient(new Uri("http://localhost:11434/"), "phi3:mini");

foreach (Embedding<float> embedding in
    await generator.GenerateAsync(["What is AI?", "What is .NET?"]))
{
    Console.WriteLine(string.Join(", ", embedding.Vector.ToArray()));
}
```

The vector can then be used to evaluate distance:

```cs
var results = await db.Documents
    .OrderBy(d => EF.Functions.VectorDistance("cosine", d.Embedding, queryVector))
    .Take(5)
    .Select(d => new
    {
        d.Id,
        d.Title,
        d.Content,
        Distance = EF.Functions.VectorDistance("cosine", d.Embedding, queryVector)
    })
    .ToListAsync();
```

There are all the different types of distance available (cosine, euclidian, manhattan)...
