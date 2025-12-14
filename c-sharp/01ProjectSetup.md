# Create projects

## Table of Contents

- [Create projects](#create-projects)
  - [Table of Contents](#table-of-contents)
  - [Basic projects](#basic-projects)
  - [Adding packages in dotnet](#adding-packages-in-dotnet)
  - [Project components](#project-components)
    - [csproj](#csproj)
    - [sln](#sln)
    - [entry point](#entry-point)
  - [Different project types](#different-project-types)
  - [C sharpier](#c-sharpier)
  - [Upgrade version of dotnet](#upgrade-version-of-dotnet)

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

## Adding packages in dotnet

The command to add packages to a dotnet project is:

```bash
dotnet add package <name of package> --version <version number>
```

Specifying a version number is very important because dotnet does not handle version compatibility but instead just checks if the latest version is compatible.

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

<https://csharpier.com/docs/About>

## Upgrade version of dotnet

```bash
dotnet tool install -g dotnet-upgrade-assistant
dotnet upgrade ./YourSolution.sln
```
