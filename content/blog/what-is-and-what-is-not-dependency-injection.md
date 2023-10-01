---
title: "What Is and What Is Not Dependency Injection"
date: 2023-10-01T19:57:08+01:00
draft: true
---
# What Is and What Is Not Dependency Injection
Have you ever been asked a question in an interview that made you pause and reflect? I recently found myself in such a situation during an interview for the role of a Senior Software Engineer. 

### "So tell me, what is Dependency Injection (DI)?" 
Dependency Injection it's a fundamental concept in software development, yet articulating a precise and clear answer can sometimes be challenging. In this blog post, we'll dive into Dependency Injection, explore what it is and what it isn't, and try to provide an ideal answer in the context of an interview.

## Dependency Injection: A 5-Cent Concept with a 25-Dollar Term
Dependency Injection, often abbreviated as DI, is a design pattern and technique that embodies the principle of Inversion of Control (IoC). IoC is all about changing the flow of control from your code to an external component or framework, which can lead to more maintainable and flexible software. But let's not get lost in jargon; as James Shore aptly puts it, ["Dependency Injection" is a "25-dollar term for a 5-cent concept."](https://www.jamesshore.com/v2/blog/2006/dependency-injection-demystified)

The core idea of Dependency Injection can be distilled into a simple statement: **"Dependency injection means giving an object its instance variables."** This quote encapsulates the essence of DI. Instead of an object creating its own dependencies, they are provided or "injected" from the outside. This concept promotes loose coupling between components and enhances testability.

To explore Dependency Injection further, let's delve into some practical examples using C#.

## Dependency Injection in C# - What It Is
Let's start with an example that demonstrates Dependency Injection:
{{< highlight csharp >}}
// Define an interface representing a dependency.
public interface ILogger
{
    void Log(string message);
}

// Create a concrete implementation of the dependency.
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

// Class that depends on ILogger through constructor injection.
public class UserService
{
    private readonly ILogger logger;

    public UserService(ILogger logger)
    {
        this.logger = logger;
    }

    public void DoSomething()
    {
        // Using the injected logger to log a message.
        logger.Log("Something happened.");
    }
}

// Configuring and using DI container (e.g., Microsoft's built-in container).
public static void Main(string[] args)
{
    var serviceProvider = new ServiceCollection()
        .AddScoped<ILogger, ConsoleLogger>()
        .AddScoped<UserService>()
        .BuildServiceProvider();

    var userService = serviceProvider.GetService<UserService>();
    userService.DoSomething();
}
{{< /highlight >}}
In this example:
1. We define an `ILogger` interface representing a dependency (logging).
2. We create a concrete implementation of the dependency (`ConsoleLogger`).
3. The `UserService` class depends on `ILogger` through constructor injection, meaning it receives an `ILogger` instance via its constructor.
4. We configure a DI container (in this case, Microsoft's built-in container) to map the dependencies.
5. We resolve the `UserService` from the container, which automatically injects the `ConsoleLogger` into its constructor when creating the instance.

## What Dependency Injection Is Not
To fully grasp the concept of Dependency Injection, it's essential to understand what it's not. Consider the following non-DI example:
{{< highlight csharp >}}
// A non-DI example where the class creates its dependencies directly.
public class UserService
{
    private readonly ILogger logger = new ConsoleLogger();

    public void DoSomething()
    {
        // Using the directly created logger.
        logger.Log("Something happened.");
    }
}
{{< /highlight >}}

In this non-DI example, the `UserService` class creates its `ILogger` dependency directly within the class. This tightly couples the class to a specific logger implementation (`ConsoleLogger`), making it less flexible and harder to test. It's essential to highlight this contrast in your interview response to demonstrate your understanding of Dependency Injection principles.

## Conclusion
Dependency Injection might have a fancy name, but at its core, it's a simple and valuable concept in software development. It's all about "giving an object its instance variables" and promoting loose coupling and testability.