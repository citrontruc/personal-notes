# Visual Studio

## Table of Content

- [Visual Studio](#visual-studio)
  - [Table of Content](#table-of-content)
  - [Libraries](#libraries)
  - [Test Explorer](#test-explorer)
  - [Debug](#debug)
  - [Profiler](#profiler)
  - [Documentation generation](#documentation-generation)
  - [Publishing projects](#publishing-projects)
  - [Viewing SQL bottlenecks](#viewing-sql-bottlenecks)

## Libraries

Visual studio has a Global usings file that imports a lot of the classic "using" statements that are used everywhere.

Visual Studio uses the nuget package manager. We can just browse for the packages we want and add them to our project.

## Test Explorer

Used to visualize all the available tests. You can run them from here.

## Debug

Using debugger mode in dotnet with f5. f10 step over (next line). f11 step into (zoom in in the class that does the operation) and shift + f11 to step out.

When error, we are in break mode, we can check the values of our variables by hovering over.

There are **diagnosis tools** in order to visualize the use of memory during the execution of our code.

## Profiler

There is a profiler which lets you see how many objects are stored in memory and helps you see if you should do stuff with garbage collector.

## Documentation generation

You can ask visual studio to automatically generate a documentation from xml comments. in order to do so, you need to go in the vuild > output and check the box indicating to generate the documentation automatically on build.

You can then specify the name of the file you want to generate.

## Publishing projects

Right click on your project and click "publish". This should let you define a publishing profile for the project.

## Viewing SQL bottlenecks

How to immediately find bottlenecks in your EF Core queries: Visualize the SQL query plan inside Visual Studio so you can immediately improve the EF Core query. How to achieve this? Use the EFCore.Visualizer Visual Studio extension.
