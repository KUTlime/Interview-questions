# Interview questions
> A repository of my practical interview questions.

## Culture & work/life balance

* Tell me about somebody who did well in my job role. What did they do?
* How many hours do you expecte me to work?
* What time do you sign off the Slack?
* Is it usually expected from me to be able to respond to Slack messages after work hours?

## C# language

### Why this code is dangerous?
```csharp
var list = collection
           .ToList()
           .ForEach(async o => await service.SendRequestAsync(o));
```

## Code simplification

### How you would simplify this code?

```csharp
var rootModel = JsonConvert.DeserializeObject<MapboxRootModel>(contentString);
if (rootModel == null || rootModel.Routes == null || !rootModel.Routes.Any())
{
    return 0;
}

return Convert.ToInt32(rootModel.Routes.First().Duration);
```

## ASP.NET Core

### App configuration

Where a service with a following configuration will log in Development mode using this code

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Configuration.AddWindowsServiceConfiguration(); // Adds appsettings.WindowsService.json
// ...
```

appsettings.json
```json
{
"Serilog": {
    "Using": [
      "Serilog.Sinks.Console"
    ],
    "WriteTo": [
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "Console",
              "Args": {
                "outputTemplate": "[...]"
              }
            }
          ]
        }
      }
    ]
  }
}
```

appsettings.Development.json
```json
{
"Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.File"
    ],
    "WriteTo": [
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "Console",
              "Args": {
                "outputTemplate": "[...]"
              }
            },
            {
              "Name": "File",
              "Args": {
                "outputTemplate": "[...]"
              }
            }
          ]
        }
      }
    ]
  }
}
```

appsettings.WindowsService.json
```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.EventLog"
    ],
    "WriteTo": [
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "EventLog",
              "Args": {
                "source": "Web API Service",
                "restrictedToMinimumLevel": "Information",
                "logName": "Application",
                "manageEventSource": true,
                "outputTemplate": "[{Timestamp:u}][{Application}][{MachineName}][{ProcessName}:{ProcessId}][{ThreadId}][{SourceContext}][{Level:u3}]{Message}{NewLine}{Exception}"
              }
            }
          ]
        }
      }
    ]
  }
}
```

## Architecture

### Automated test design for a console application

Imagine that you are writting a non-trivial console application and you want to write automated tests. We can call them unit/integration tests if you like.

The application would use a class called `ConfigProviderClient` that comes from 3rd party library. You don't have an access to the source code (_besides decompilation_) and **you cannot alter the source ode of `ConfigProviderClient`**.

In class `ConfigProviderClient`, you will be calling a method `AddConfigurationAsync(...)`. It is obvious that you would be calling some 

Here is a structure of the `ConfigProviderClient` class and related objects.

```csharp
public record ApiConfiguration(Uri ConfigProviderUri, string ServiceName, string Token);

public record Request(string Key, string Value, bool Encrypted);

public class ConfigProviderClient
{
    private readonly ApiConfiguration _configuration;

    public ConfigProviderClient(ApiConfiguration configuration) => _configuration = configuration;

    public async Task<bool> AddConfigurationAsync(Request request)
    {
        // ...
        // not important implementation details
        // ...
        // Calling of external API via HTTP client 
    }
}
```
Your console application must collect arguments from the command line, read a content of some JSON file, do some transformation, validation and create a correct request for the external API that is called inside of `ConfigProviderClient`, in method `AddConfigurationAsync(...)`.

The question is: **How do you ensure that you can write automated tests that verify that your application works correctly and the request is built correctly?**

I'm not interested in any particular implementation, I'm interested how do you manage the situation that you are calling an external API in a test.

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

## Answers

### Why this code is dangerous?
```csharp
var list = collection
           .ToList()
           .ForEach(async o => await service.SendRequestAsync(o));
```
The method signature is async void. This will swallow all exceptions that might happend in `SendRequestAsync` calls.

If this would be in tests, they would be false possitive.

### App configuration

Since the configuration object is a dictionary where number of keys matters, the logs will be logged to
* Console for Production | Staging
* Console & File for Development
* EventLog & File for Windows Service, because the console sink will be overriden since it is on [0] position.

### Automated test design for a console application

A decorator design pattern around `ConfigProviderClient`, your own interface that will wrap the `ConfigProviderClient` class.

### How you would simplify this code?

```csharp
var rootModel = JsonConvert.DeserializeObject<MapboxRootModel>(contentString);
if (rootModel == null || rootModel.Routes == null || !rootModel.Routes.Any())
{
    return 0;
}

return Convert.ToInt32(rootModel.Routes.First().Duration);
```
This could be a oneliner

```csharp
return rootModel?.Routes is null or [] ? 0 : Convert.ToInt32(rootModel.Routes.First().Duration);
```


