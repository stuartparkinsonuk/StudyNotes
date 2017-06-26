# ASP.net Core Web Application

## Program.cs
`Program.cs` contains the `Main` method which is the application starting point.

This uses `WebHostBuilder` to build a web hosting app using extension methods. The default `Program.cs` file created by the new project template is shown below.

``` c#
 public class Program
    {
        public static void Main(string[] args)
        {
            var host = new WebHostBuilder()
                .UseKestrel()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseIISIntegration()
                .UseStartup<Startup>()
                .UseApplicationInsights()
                .Build();

            host.Run();
        }
    }
```

`.UseKestrel()` - use the Kestrel web server (can change to use others) 
`.UseContentRoot(Directory.GetCurrentDirectory())` - tells the server where the root directory for content is 
`UseIISIntegration()` - 
`UseStartup<Startup>()` - tells server which class to use for startup configuration (in this case the `Startup` class defined in `Startup.cs`)

`UseApplicationInsights()` -

`Build();` - now go and build us a webserver that we can run with `host.Run();`


## Startup.cs
`Startup.cs` defines the request handling pipeline, and must contain the following methods:

``` c#
public void ConfigureServices(IServiceCollection services) {...}
public void Configure(IApplicationBuilder app) {...}
```
