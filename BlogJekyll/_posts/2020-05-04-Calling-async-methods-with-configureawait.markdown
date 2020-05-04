---
layout: post
title:  "Calling async methods with ConfigureAwait"
date:   2020-05-04 00:00:00 +0100

tags: Async
---

When I started using SonarQube/SonarLint to improve code quality it made me aware of using the ConfigureAwait method available on any async method call. The linting rule defines:

> After an `await`ed `Task` has executed, you can continue execution in the original, calling thread or any arbitrary thread. Unless the rest of the code needs the context from which the `Task` was spawned, `Task.ConfigureAwait(false)` should be used to keep execution in the `Task` thread to avoid the need for context switching and the possibility of deadlocks.
>
> This rule raises an issue when code in a class library `await`s a `Task` and continues execution in the original calling thread.
>
> [quoted from: sonarsource.com](https://rules.sonarsource.com/csharp/RSPEC-3216)

In this post, I will take little dive into asynchronous programming and refer to relevant resources to look at. After that, I will explain the usage of the ConfigureAwait method.

**Table of contents:**
* Table of Contents
{:toc}

### What is asynchronous programming

If you are not very familiar with asynchronous programming I would advise you to look into the high-level explanations found in the Microsoft Docs articles below.

- [Asynchronous programming with async and await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
- [Async in depth](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth)


My summary for these articles is:

A single method call chain in your program when called synchronously is executed on 1 thread. If the code has to wait for an external call handled by the operating system (accessing a disk or network resource) that single thread will stay idle and wait for that external call to finish then resume its work.  

When using asynchronous programming you allow the call chain execution to be paused and free up the thread for other work while waiting for the response of an external or long-running call. This allows the software or system that runs the software to complete more work and utilize it’s resources more efficiently.  

Another great article on Microsoft Docs is "https://docs.microsoft.com/en-us/dotnet/csharp/async" that explains how to identify situations in which asynchronous programming is especially helpful. Also, be aware of the ["Important info and advice](https://docs.microsoft.com/en-us/dotnet/csharp/async#important-info-and-advice)" section in that article for some extra tips even if you already know and use asynchronous programming.  

### The Task-based Asynchronous Pattern (TAP) 

.NET Framework 4 and above (including .NET Core) use the Task-based Asynchronous Pattern (TAP) for writing asynchronous code. There [are in-depth and concrete articles for Task-based asynchronous pattern (TAP)](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) on how to use the Task-based Asynchronous Pattern in C#. You can also find explanations on the "Legacy" asynchronous patterns used in the .NET framework using the left-hand content menu.

The "[Consuming the Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern)" contains an explanation on how to consume asynchronous methods. This information is good to know as more and more methods of the .NET framework are starting to become asynchronous methods. This article also contains [an explanation for the ConfigureAwait method](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern#configuring-suspension-and-resumption-with-yield-and-configureawait).

### Using ConfigureAwait 

As stated in the last paragraph the section in "[Consuming the Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern)" explaining the ConfigureAwait is "[Configuring Suspension and Resumption with Yield and ConfigureAwait](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern#configuring-suspension-and-resumption-with-yield-and-configureawait)".

It explains that when resuming work after an await, the default behavior is to resume work on the original thread that was paused. If you do not explicitly use the ConfigureAwait method the default behavior will be the same as ConfigureAwait(true).

When using asynchronous methods in a web application this default behavior allows you to retain the value certain variables like for example the HttpContext variable after an awaited method. 

If you have no need for these values (or capture all the values on entry and hand them off to your asynchronous code) you can use ConfigureAwait(false). This will allow the continuation of execution on any thread and allows the system to improve its scheduling of work on the available threads. As stated in the SonarQube rule listed at the start of the article this may also prevent certain deadlock scenarios.

However, be aware that when introducing ConfigureAwait(false) that you may lose some context variables and get some unexplainable null reference exceptions if the code is expecting those variables to still have a value. 

There is a great [Microsoft developer blogpost](https://devblogs.microsoft.com/dotnet/configureawait-faq/ ) with extra useful information and a FAQ section about the usage of ConfigureAwait. It has a more in-depth explanation of how to use ConfigureAwait. 