# Visual Studio

## Table of Content

- [Visual Studio](#visual-studio)
  - [Table of Content](#table-of-content)
  - [Libraries](#libraries)
  - [Test Explorer](#test-explorer)
  - [Debug](#debug)
  - [Profiler](#profiler)

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
