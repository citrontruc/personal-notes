# Parallel

## Table of content

- [Parallel](#parallel)
  - [Table of content](#table-of-content)
  - [Why?](#why)
  - [The Parallel library](#the-parallel-library)
  - [Error Handling](#error-handling)

## Why?

If you have a lot of tasks to run, you may want to process them in parallel and not all one after another. You could wrap them up in a task.Run and run them all like that but it would involve quite a bit of code and handling. A better way would be to use the Parallel library.

## The Parallel library

Careful, the Parallel library blocks the calling thread ==> it is a powerful tool that needs to be used with care. You can do parallel in a Task.Run() to do that in new threads. Will have less threads available in this case.

Parallel executes them in the most optimized way. It could be all in parallel or just the best way depending on your hardware.

```cs
Parallel.Invoke(task1, task2);
```

There are other functions with the Parallel library like Parallel.For() and Parallel.ForEach().

If you need, there are a lot of options that you can tinker (max degrees of parallelism, cancellation token)...

## Error Handling

If a task failed, we start by running all the tasks and then we get the exception at the very end. Can be tricky to detect an error.

If you want to break a loop and make it stop, there is a state parameter for that. Have a look if necessary.
