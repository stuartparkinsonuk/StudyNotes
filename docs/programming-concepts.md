# Programming Concepts
---

## Dependency Injection / Inversion of Control

[Introduction to Dependency Injection in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection) - Microsoft page

### What and Why?

> A **dependency** is another class that the code you are writing depends on.

Say you are writing an app and you want to log certain performance information - maybe you get current weather data from a web service and you want to log how long each request takes. You write a class to do that logging and call it `Logger` with a `Log(string message)` method which writes your message to a file.

``` c#
public class Logger
{
	public void Log(string message)
	{
		// write message to log file
	}
}
```

Here is the class which gets the weather data and logs the time taken. 

``` c#
public class MyClass
{
	private Logger logger = new Logger();

	public void GetWeather()
	{
		Stopwatch sw = new Stopwatch();
		sw.Start();
		// Get weather info from web service ...
		sw.Stop();
		logger.Log($"Webservice time = {sw.Elapsed}");
	}
}
```

Your `MyClass` object now **depends** on the `Logger` class. The `Logger` class is hard-coded into you program. We say that there is 'tight coupling' between the `MyClass` and the `Logger` classes.

Now this is all well and good for a small app that is not going to change, but has several drawbacks.

1. It is not easy to test `MyClass` without having to open up the log file to see what was actually written to it.
1. It is not easy to change the logger if we decide that instead of logging to a file we want to log to a database instead.

It would be nice if we could just write `MyClass` so that it could use any logger object that we decided to give it. The we could use a test logger for testing, another for logging to file, and another for writing the messages to a database, and so on. It would be really nice if, when we wrote `MyClass`, we didn't need to know anything about the logger that would ultimately be used other than all we had to do was pass it a message.

The problem, of course, is the line

`private Logger logger = new Logger();`

To write this line of code we need a reference to the `Logger` class dependency, and once we have written it we can't change the logger without changing the `MyClass` source code. Instead of hard-coding the dependancy, we would like to be able to pass the dependency in to `MyClass` when we run our code.

### Interfaces

An [Interface](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface) difines the signatures of methods, properties, events or indexers of a class. A class or struct that implements the interface must implement the members of the interface that are specified in the interface definition.


