# Logging

## Table of content

- [Logging](#logging)
  - [Table of content](#table-of-content)
  - [Why log?](#why-log)
  - [Best Practices](#best-practices)
    - [Log levels](#log-levels)
    - [Log solutions in dotnet](#log-solutions-in-dotnet)
    - [Code](#code)
  - [Middleware](#middleware)

## Why log?

Debugging, performance monitoring (log execution times, memory usage & other) and security auditing. A log has to have one of these 3 uses or else it is useless.

We want to avoid useless, scattered logs without any context.

## Best Practices

- Have multiple logging levels in order to avoid lack of visibility.
- Delete old logs.
- Consolidate logs when you need to know how your app / api is used.

### Log levels

- Trace: Log calls. and states. We want to evaluate performances. Only in development.
- Debug: Only on development. We see the state of things.
- Information: Things happen. Helps to know what is happening in the application.
- Warning: something unexpected happened but we have not reached a point of failure. Better do something about it though.
- Error: something failed but we can recover. Most often, use with try-catch.
- Critical / failure: Something catastrophic happened (example: can't connect to key service). Critical happen just before the system fails.

Examples:

```cs
var logger = LoggerFactory.Create(builder => builder.AddConsole()).CreateLogger();
logger.LogTrace("Entering method ProcessPayment with parameters: {amount}, {currency}", amount, currency); // Most information. Gives current state of things.
logger.LogDebug("User {userId} fetched {itemCount} items from the database", userId, itemCount);
logger.LogWarning("Disk space running low: {availableSpace}MB remaining", availableSpace); // Use for deprecation, timeouts, slow response time...
```

### Log solutions in dotnet

- Serilog (comfortable. You have multiple log levels and you can log to files).
- Nlog (close to serilog, maybe a little less mature).
- Microsoft.Extensions.Logging (Easy to use. Does not need any additional configuration). Does not store logs to file.
- Log4net (you need a config file). Stable and mature but not very dotnet specific. Starting to age.
- ZLogger (New modern library supposed to be very efficient). lacks some functionalities right now.

### Code

Example for microsoft logging:

```cs
using Microsoft.Extensions.Logging;

using ILoggerFactory factory = LoggerFactory.Create(builder => builder.AddConsole());
ILogger logger = factory.CreateLogger("Program");
logger.LogInformation("Hello World! Logging is {Description}.", "fun");
```

Example code for serilog.

```cs
using System;
using Serilog;
using Serilog.Events;
using Serilog.Formatting.Json;

namespace SerilogAdvanced
{
    class Program
    {
        static void Main(string[] args)
        {
            using var log = new LoggerConfiguration()
                             .WriteTo.Console()
                             .WriteTo.File(new JsonFormatter(),
                                           "app.json",
                                           restrictedToMinimumLevel: LogEventLevel.Warning)
                             .WriteTo.File("all-.logs",
                                           rollingInterval: RollingInterval.Day)
                             .MinimumLevel.Debug()
                             .CreateLogger();
            log.Information("Hello, Serilog!");
            log.Warning("Warning, Serilog!");
        }
    }
}
```

NLOG example:

```cs
using NLog;
using NLog.Common;
using NLog.Config;
using NLog.Targets;
using NLog.Layouts;

namespace LoggerExample
{
    class Program
    {
        private static Logger logger = LogManager.GetCurrentClassLogger();

        static void Main(string[] args)
        {
            // Create logging configuration
            var config = new LoggingConfiguration();

            // Define file target with JSON layout
            var fileTarget = new FileTarget { FileName = "myApp.log" };
            fileTarget.Layout = new JsonLayout
            {
                Attributes = {
                    new JsonAttribute("timestamp", "${date:format=yyyy-MM-ddTHH:mm:ssZ}"),
                    new JsonAttribute("level", "${level}"),
                    new JsonAttribute("message", "${message}"),
                    new JsonAttribute("exception", "${exception:format=Message}")
                }
            };
            config.AddTarget("file", fileTarget);

            // Define console target
            var consoleTarget = new ConsoleTarget();
            config.AddTarget("console", consoleTarget);

            // Create logging rules
            var fileRule = new LoggingRule("*", LogLevel.Info, fileTarget);
            config.LoggingRules.Add(fileRule);

            var consoleRule = new LoggingRule("*", LogLevel.Debug, consoleTarget);
            config.LoggingRules.Add(consoleRule);

            // Set configuration
            LogManager.Configuration = config;

            // Log messages
            logger.Info("Hello from Nlog");
            logger.Error("Error from Nlog");
        }
    }
}
```

## Middleware

You can put logs wherever you want but if you want to log all of your requests and have them be saved to log files, include your logger to the middleware.
