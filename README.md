<h1 align='center'>SimpleDi</a>

## Declarative Dependency Injection and Configuration for .NET

SimpleDi allows you to inject interfaces and types using attributes. No need for complex frameworks or manually adding injections to your startup code.

You can also put service configuration and web application builder setup code in classes, allowing your class libraries to automatically be part of these processes.

## Setup
```cs
using DigitalRuby.SimpleDi;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSimpleDi();

// for web apps (not needed for non-web apps):
var host = builder.Build();
host.UseSimpleDi();
```

## Assembly Scanning
By default, only assemblies with names prefixed with the first part of your entry assembly name will be  included for memory optimization purposes. You can change this in the `AddSimpleDi` and `UseSimpleDi` by passing a regex string to match assembly names.

## Implementation

Create a class, `MyInterfaceImplementation`
```cs
// Interface will be available in constructors
public interface IMyInterface
{
}

// bind the class to simpledi, implementing IMyInterface
[Binding(BindingType.Singleton)]
public sealed class MyInterfaceImplementation : IMyInterface
{
}
```

Inject the interface into the constructor of another class:
```cs
[Binding(BindingType.Singleton)]
public sealed class MyClass
{
	public MyClass(IMyInterface myInterface)
	{
	}
}
```
## Hosted Services

```cs
[Binding(BindingType.Singleton)]
public sealed class MyHostedClass : IHostedService
{
	public Task StartAsync(CancellationToken cancellationToken) => Task.CompletedTask;
	public Task StopAsync(CancellationToken cancellationToken) => Task.CompletedTask;
}
```
## Select Interfaces
```cs
// only register MyClass concrete type
[Binding(BindingType.Singleton, null)]
public sealed class MyClass1 : IInterface1, IInterface2
{
}

// only register MyClass concrete type and IInterface1
[Binding(BindingType.Singleton, typeof(IInterface1))]
public sealed class MyClass2 : IInterface1, IInterface2
{
}
```
When using the `BindingAttribute` you can specify an optional conflict resolution:
`public BindingAttribute(ServiceLifetime scope, ConflictResolution conflict, params Type[]? interfaces)`

```cs
/// <summary>
/// What to do if there is a conflict when registering services
/// </summary>
public enum ConflictResolution
{
    /// <summary>
    /// Add. This will result in multiple services for an interface if more than one are added.
    /// </summary>
    Add = 0,

    /// <summary>
    /// Replace. This will make sure only one implementation exists for an interface.
    /// </summary>
    Replace,

    /// <summary>
    /// Skip. Do not register this service if another implementation exits for the interface.
    /// </summary>
    Skip
}
```

## Configuration

Given a json file in your project `config.json` (set as content and copy newer in properties):
```json
{
  "DigitalRuby.SimpleDi.Tests.Configuration":
  {
    "Value": "hellothere"
  }
}
```

And a class with the same namespace as in the file...

```cs
namespace DigitalRuby.SimpleDi.Tests;

/// <summary>
/// Config class that binds to key DigitalRuby.SimpleDi.Tests.Configuration
/// </summary>
[Configuration]
public sealed class Configuration
{
    /// <summary>
    /// Example value
    /// </summary>
    public string Value { get; set; } = string.Empty; // overriden from config
}
```

You can inject your `Configuration` class into any constructor as normal.

Make sure to add your config file to your configuration builder:

```cs
var builder = WebApplication.CreateBuilder();
builder.Configuration.AddJsonFile("config.json");
```

You can create multiple keys in your configuration file for each class annotated with the `Configuration` attribute, or use separate files.

Instead of custom files you can also just use your `appsettings.json` file, which is added by .NET automatically.

## Service setup code
You can create classes in your project or even class library to run service setup code. Simply inherit from `DigitalRuby.SimpleDi.IServiceSetup` and provide a private constructor:
```cs
internal class ServiceSetup : IServiceSetup
{
    private ServiceSetup(IServiceCollection services, IConfiguration configuration)
    {
        // perform service setup
    }
}
```

## Web app setup code
Similar to service setup code, you can create classes to run web app setup code:
```cs
internal class AppSetup : IWebAppSetup
{
    private AppSetup(IApplicationBuilder appBuilder, IConfiguration configuration)
    {
        // perform app setup
    }
}
```

---

Thank you for visiting!

-- Jeff

https://www.digitalruby.com
