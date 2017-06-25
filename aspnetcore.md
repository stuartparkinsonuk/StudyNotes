#Startup

```Program.cs``` contains the ```Main``` method which is the application starting point.
This uses ```WebHostBuilder``` to build a web hosting app using extension methods, eg.

```C#
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
