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
public class WeatherForecast
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

Your `WeatherForecast` object now **depends** on the `Logger` class. The `Logger` class is hard-coded into your program. We say that there is 'tight coupling' between the `WeatherForecast` and the `Logger` classes.

Now this is all well and good for a small app that is not going to change, but has several drawbacks.

1. It is not easy to test `WeatherForecast` without having to open up the log file to see what was actually written to it.
1. It is not easy to change the logger if we decide that instead of logging to a file we want to log to a database instead.

It would be nice if we could just write `WeatherForecast` so that it could use any logger object that we decided to give it. The we could use a test logger for testing, another for logging to file, and another for writing the messages to a database, and so on. It would be really nice if, when we wrote `WeatherForecast`, we didn't need to know anything about the logger that would ultimately be used other than all we had to do was pass it a message.

The problem, of course, is the line

`private Logger logger = new Logger();`

To write this line of code we hard code a reference to the `Logger` class, and once we have written it we can't change the logger without changing the `WeatherForecast` source code. Instead of hard-coding the dependancy, we would like to be able to pass the dependency in to `WeatherForecast` when we run our code.

### Interfaces

> An [Interface](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface) defines the signatures of methods, properties, events or indexers of a class. A class or struct that implements the interface must implement the members of the interface that are specified in the interface definition.

An interface defines what a class will look like 'from the outside'. If a class implements a particular interface, you know that it will have the exact methods and properties etc. that are defined in the interface, with those exact parameters and types and return values. What goes on inside the class is none or your concern - what is important is that you know the interface and so you can use it in your program.

Several different implementations of the interface might perform the task in completely different ways. For example, we could have one implementations of a logger interface which logs to file and another which logs to a database, but because they look exactly the same from the outside they can be used interchangeably in your program.

First we need to define an interface that describes the essential public methods of our logger.

``` c#
public interface ILogger
{
	void Log(string message);
}
```

This interface dictates that anything that impements it must provide a `Log` method which takes a string as a parameter and returns a void.

Now we can write a logger class which implements this interface.

``` c#
public class FileLogger : ILogger
{
	public void Log(string message)
	{
		// code to write message to file
	}
}
```

### Constructor Injection

One simple form of dependency injection is 'Constructor Injection'. We pass the dependency that we want the class to use into to class through its constructor at the time we instantiate an instance of the class.

Here is a modified version of our class which takes a logger object in its constructor, keeps a reference to it in a private variable, and uses that logger when it needs to log something.

``` c#
public class WeatherForecast
{
	private ILogger logger;

	public WeatherForecast(ILogger specific_Logger_to_Use)
	{
		logger = specific_Logger_to_Use;
	}

	public void GetWeather()
	{
		Stopwatch sw = new Stopwatch();
		sw.Start();
		// Get weather info from web service ...
		Thread.Sleep(1000);
		sw.Stop();
		logger.Log($"Webservice time = {sw.Elapsed}");
	}
}
```

Note in line 3 that our private variable is of type `ILogger`. We don't need to know which specific type of logger we are eventually going to get, just so long as it implements `ILogger`.

When we instantiate our class we can pass it a file logger, an email logger, or any other logger that implements the `ILogger` interface. We can even make a mock logger that we can use during testing.

``` c#
class Program
{
	static void Main(string[] args)
	{
		// this time we are going to use a FileLogger
		FileLogger fileLogger = new FileLogger();

		// create a new instance of WeatherForecast and pass it the FileLogger to use
		WeatherForecast forecast = new WeatherForecast(fileLogger);

		forecast.GetWeather();
	}
}
```

Now `WeatherForecast` does not have to know which specific logger it is going to use - it only cares that when we come to use it we pass it a logger object that implements the `ILogger` interface. In this program we are giving it a `FileLogger`, but in another program we might give it a `DatabaseLogger` and we wouldn't have to change `WeatherForecast` at all.

This is good because we can write `WeatherForecast`, test it, and tuck it away in a library somewhere and never have to touch it again. We can use it in lots of places, and each time we use it we can decide at the time which logger we want `WeatherForecast` to use.

We have 'injected' the dependency into `WeatherForecast` at the time we instantiated it.

### Inversion of Control

IoC ([Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html)) is a general programming principle where the flow of control through the program (which would normally have been under 'your' control) is 'inverted' so it is now under the control of a framework.

For example, in traditional text based programming (such as a batch file or a console program) you might write something to get the users name and greet them.

``` c#
static void Main(string[] args)
{
	Console.Write("Enter your name: ");
	string name = Console.ReadLine();
	Console.WriteLine($"Welcome, {name}");
}
```

The flow of the program is enitely under your control. You decide when to output to the console, and you decide when to read input from the console. It is your code that is running, and you make calls into other code (such as `Console.WriteLine`) when you need to.

However, if you are using some kind of framework (Windows forms, ASP.NET, Android, etc.) you might have a form with a text entry box labelled 'Enter your name', a button labelled 'Next', and an empty label called response. Your code would look something like

``` c#
private void NextButton_Click(object sender, RoutedEventArgs e)
{
	Response.Text = $"Welcome, {NameEntry.Text}";
}
```

Now it is *not* your code that is in control - the *framework* is in control. When the user runs the application, it is the framework code that is running, and when the user hits the 'Next' button the framework makes a call into your code through the event handler mechanism.

You have handed control off to the framework and told it, "Call me when you need me to do something." This is also known as the [Hollywood Principle](http://deviq.com/hollywood-principle/) - "Don't call us. We'll call you!"

There are many forms that this inversion of control can take, but an important factor in all of them is that the framework code is already written and you don't want to be messing with it. Somehow you need to be able to get the framework to call your code when the framework designer, at the time the framework was written, had no idea that your code even existed (in fact, it probably didn't).

Registering for events is one programming pattern which can achieve this. In the code above we have (elsewhere) told the framework which method to call when the button gets pushed. (Often this hookup is done behind the scenes by the graphical designer you use to create your form). When the button is pushed, the framework calls the `NextButton_Click` method to execute *our* code. The only stipulation is that the `NextButton_Click` method conforms to what the framework expects by way of parameters and return type for an event handler.

Dependency Injection is just another method of Inversion of Control, where your code (say, the logger) gets called by some other code (maybe the framework, maybe some other bit of your code) that itself knows nothing about your logger. All you need to do is follow the rules (ie. implement the `ILogger` interface) and you can get the other code to call your code whenever it needs to.

## IoC Containers

We could just stop there, because with Dependency Injection we have all we need to 'decouple' (ie. make independent) parts of our code. We can write a `GetWeather` class which uses a logger, and we don't care which logger is used when we write the `GetWeather` class (as long as it implements `ILogger`). And we can write lots of other classes that use loggers, and write lots of different loggers, and mix and match to our heart's content. And many applications do just use manual Dependency Injection like this, where in the final application the desired logger service is supplied to each particular consumer class.

However, we can go another step in automating this process by using what is called an IoC Container.

The code behind an IoC Container (sometimes also called a DI Container) can get a little complex and we are not going to delve into it at all in this article. There are [many ready-made IoC Containers](https://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx) available, and ASP.NET Core includes a simple one as part of the framework which is used so that *you* can tell the *framework* what you want it to use as (for example) a logger. You can pick one of the many existing classes or make your own. You can also use the IoC Container to handle dependency injection for your own code.

### What Problem are we Solving?

Earlier we used this code to instatiate a logger of our choice and pass it to `WeatherForecast`

``` c#
// this time we are going to use a FileLogger
FileLogger fileLogger = new FileLogger();

// create a new instance of WeatherForecast and pass it the FileLogger to use
WeatherForecast forecast = new WeatherForecast(fileLogger);
```

There are two possible problems with this.

1. We might use this bit of code (or a bit very similar) in many places. If we wanted to change loggers to use the database logger instead we would have a lot of places to go and change our code.
2. The above example is very simple because although the `WeatherForecast` class has a dependency on a logger, the logger does not have a dependency on anything else. This is often not the case. Maybe our database logger needs a database object in it's constructor (and there may be several different types of database we could pass it), and each database object needs a connection string passed into it's constructor. Quite soon we end up with a complex 'dependency graph' - a large tree of objects depending on objects depending on objects.

In the code above, the line

``` c#
WeatherForecast forecast = new WeatherForecast(fileLogger);
```

is where we instantiate our `WeatherForecast` object. The logger we are using is hard coded into this line which is inflexible and hard to maintain, especially where either of the above two points apply.

### The Solution

An IoC Container is a 'factory' which instantiates instances of classes for us, and in doing so also instantiates instances of all the dependencies that those classes need and injects them through the constructors. There are various different ways that such containers can be implemented, but the following is typical.

First we create an instance of a container, and then we 'register' all the dependencies with it.

``` c#
IOContainer iocContainer = new IOCContainer();

iocContainer.AddService<ILogger, FileLogger>();
```

This code registers the concrete type `FileLogger` and tell the container to use a `FileLogger` wherever it needs a concrete implementation of `ILogger`.

Then we can use the container to create an instance of our `WeatherForecast` class as follows.

``` c#
WeatherForcast forecast = iocContainer.Resolve<WeatherForecast>();
```

The container looks at the `WeatherForecast` class we have asked for and sees that in its constructor it needs an instance of an `ILogger`. It looks at what has been registered and sees that for an `ILogger` it needs to create an instance of the `FileLogger` class, which it does. It can now create an instance of the `WeatherForecast` class, passing the `FileLogger` to its constructor.

When we want to change the logger, we only have to change it in the one place where we regisered it with the container, which is normally done right at the beginning of the program before anything else.

Also, if there is a deep tree of nested dependencies (which is quite normal in many programs) the container will create instances of all the dependencies that are needed - all we need to do is register them.

## Implementation in ASP.Net Core

---



