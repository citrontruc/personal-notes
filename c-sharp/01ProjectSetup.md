# Create projects

## Basic projects

```bash
dotnet new console # Creates new project. Creates a csproj file, obj and bin files.
dotnet new sln -n "name of project"
dotnet sln add Game/snake-online-prototype.csproj
dotnet run to run project.
dotnet add package Raylib-cs
dotnet new gitignore
dotnet new tool-manifest
```

Automatic formatting for vs-code:
on windows, shift + alt + f,
linux ctrl + shift + I

```bash
dotnet format //deletes whitespaces. Linter.
dotnet run --project <name of project if you have multiple projects in you sln>
```

## Different project types

- console (basic with nothing)
- webapi (minimal api)
- webapp or mvc (to create apps)
- blazorserver and blazorwasm for web assembly (frontend for webapps)

## What you need for a project

You need a class with a static main. It can take arguments which will then be given when using dotnet run.

```bash
dotnet run -- 10 20
```

## C sharpier
```bash
dotnet tool install csharpier
dotnet csharpier format .
```
https://csharpier.com/docs/About

## Tests
```bash
dotnet new mstest
dotnet sln add <path to project>
dotnet add reference <path to project>
```
https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-mstest
