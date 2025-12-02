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

## Project components

### csproj

Role: Defines the project's identity, dependencies, target framework, compilation settings, and files to be included.

### sln

The .sln file, or Solution file, is a logical container that organizes one or more related projects (which are defined by their .csproj files). Role: Organizes, manages, and builds a set of projects as a unified application or system. It does not contain code itself.

### entry point

Can be a static main but can also be just a top level program. By convention, we call it main.

If you want to give args:

```bash
dotnet run -- 10 20
```

## Different project types

- console (basic with nothing)
- webapi (minimal api)
- webapp or mvc (to create apps)
- blazorserver and blazorwasm for web assembly (frontend for webapps)

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
