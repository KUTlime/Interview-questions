# Interview questions

> A repository of my practical interview questions.

## Culture & work/life balance

* Tell me about somebody who did well in my job role. What did they do?
* How many hours do you expect me to work?
* What time do you sign off Slack?
* Is it usually expected of me to be able to respond to Slack messages after work hours?

## .NET Pell-Mell

* How many projects have you done in .NET 9/8/7/6?
* Do you know the Mediator pattern?
* Do you know Roslyn source-code generators? Have you used them somehow before?

## C# language

### Why is this code dangerous?
```csharp
var list = collection
           .ToList()
           .ForEach(async o => await service.SendRequestAsync(o));
```

## Code simplification

### How would you simplify this code?

```csharp
var rootModel = JsonConvert.DeserializeObject<MapboxRootModel>(contentString);
if (rootModel == null || rootModel.Routes == null || !rootModel.Routes.Any())
{
    return 0;
}

return Convert.ToInt32(rootModel.Routes.First().Duration);
```

### Can this code be simplified?

```csharp
public static async Task UpdateActionCompletionTime(IRidesRepository ridesRepository, UpdateActionCompletionTimeCommand command, CancellationToken cancellationToken)
{
    var ride = await ridesRepository.GetRideById(command.RideId, cancellationToken);
    var rideRequestId = ride?.Request.Id ?? throw new ArgumentNullException($"Ride '{ride?.Id}' does not have a mandatory ride request.", nameof(ride.Request));
    switch (command.RideStatus)
    {
        case RideStatus.Ongoing:
            await SetActionCompletionTime(
                ridesRepository,
                command.ActionCompletionTime,
                x => x.PickUps != null && x.PickUps.Contains(ride.Request.Id),
                cancellationToken);
            break;
        case RideStatus.Finished:
            await SetActionCompletionTime(
                ridesRepository,
                command.ActionCompletionTime,
                x => x.DropOffs != null && x.DropOffs.Contains(ride.Request.Id),
                cancellationToken);
            break;
        case RideStatus.New:
        case RideStatus.ConfirmedByDriver:
        case RideStatus.Boarding:
        case RideStatus.Canceled:
        case RideStatus.Rejected:
            break;
        default:
            throw new ArgumentOutOfRangeException(nameof(command.RideStatus), command.RideStatus, "Ride status is outside of mapping range");
    }
}
```

Can you simplify this code?

```csharp
[SuppressMessage("Maintainability", "AV1500:Member or local function contains too many statements", Justification = "A test.")]
public ValidateOptionsResult Validate(string? name, BuildTemplatesOptions options)
{
    if (!string.Equals(options.WindowsServer2022.Version, _config?.WindowsServer2022.Version, StringComparison.OrdinalIgnoreCase))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 version is invalid.");
    }

    if (!string.Equals(options.WindowsServer2019.Version, _config?.WindowsServer2019.Version, StringComparison.OrdinalIgnoreCase))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2019 version is invalid.");
    }

    if (!string.Equals(options.WindowsServer2016.Version, _config?.WindowsServer2016.Version, StringComparison.OrdinalIgnoreCase))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2016 version is invalid.");
    }

    if (options?.WindowsServer2022?.Version is null)
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 version is required.");
    }

    if (options?.WindowsServer2022?.Release is null)
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 release is required.");
    }

    if (options?.WindowsServer2022?.TaskSequence is null)
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 task sequence is required.");
    }

    if (IsInvalidBuildTemplateOption(options?.WindowsServer2022))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 task sequence is required.");
    }

    if (IsInvalidBuildTemplateOption(options?.WindowsServer2019))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 task sequence is required.");
    }

    if (IsInvalidBuildTemplateOption(options?.WindowsServer2016))
    {
        return ValidateOptionsResult.Fail("The Windows Server 2022 task sequence is required.");
    }

    return ValidateOptionsResult.Success;
}
```

## ASP.NET Core

### App configuration

Where a service with the following configuration will log in the Development mode using this code

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

Imagine that you are writing a non-trivial console application and you want to write automated tests. We can call them unit/integration tests if you like.

The application would use a class called `ConfigProviderClient` that comes from a third-party library. You don't have access to the source code (_besides decompilation_), and you cannot alter the source code of `ConfigProviderClient`.

In class `ConfigProviderClient`, you will call the `AddConfigurationAsync(...)` method. You would be calling some 

Here is a structure of the `ConfigProviderClient` class and related objects.

```csharp
public record ApiConfiguration(Uri ConfigProviderUri, string ServiceName, string Token);

public record Request(string Key, string Value, bool Encrypted);

public class ConfigProviderClient(ApiConfiguration configuration)
{
    public async Task<bool> AddConfigurationAsync(Request request)
    {
        // ...
        // not important implementation details
        // ...
        // Calling of external API via HTTP client 
    }
}
```
Your console application must collect arguments from the command line, read the content of a JSON file, do a transformation, validation, and create a correct request for the external API that is called inside of `ConfigProviderClient`, in method `AddConfigurationAsync(...)`.

The question is: **How do you ensure that you can write automated tests that verify that your application works correctly and the request is built correctly?**

I'm not interested in any particular implementation; I'm interested in how you manage the situation where you are calling an external API in a test.

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

### Why is this code dangerous?
```csharp
var list = collection
           .ToList()
           .ForEach(async o => await service.SendRequestAsync(o));
```
The method signature is async void. This will swallow all exceptions that might happen in `SendRequestAsync` calls.

If this were in tests, they would be false positives.

### App configuration

Since the configuration object is a dictionary where the number of keys matters, the logs will be logged to
* Console for Production | Staging
* Console & File for Development
* EventLog & File for Windows Service, because the console sink will be overridden since it is in [0] position.

### Automated test design for a console application

A decorator design pattern around `ConfigProviderClient`, your interface that will wrap the `ConfigProviderClient` class.

### How would you simplify this code?

```csharp
var rootModel = JsonConvert.DeserializeObject<MapboxRootModel>(contentString);
if (rootModel == null || rootModel.Routes == null || !rootModel.Routes.Any())
{
    return 0;
}

return Convert.ToInt32(rootModel.Routes.First().Duration);
```
This could be an oneliner

```csharp
return rootModel?.Routes is null or [] ? 0 : Convert.ToInt32(rootModel.Routes.First().Duration);
```

### Can this code be simplified?

```csharp
public static async Task UpdateActionCompletionTime(UpdateActionCompletionTimeCommand command, CancellationToken cancellationToken)
{
    var ride = await _ridesRepository.GetRideById(command.RideId, cancellationToken);
    ArgumentNullException.ThrowIfNull(ride?.Request.Id); // We can use this handy method
    switch (command.RideStatus)
    {
        case RideStatus.Ongoing:
            await SetActionCompletionTime(
                ridesRepository,
                command.ActionCompletionTime,
                x => x.PickUps is not null && x.PickUps.Contains(ride.Request.Id),
                cancellationToken);
            break;
        case RideStatus.Finished:
            await SetActionCompletionTime(
                ridesRepository,
                command.ActionCompletionTime,
                x => x.DropOffs is not null && x.DropOffs.Contains(ride.Request.Id),
                cancellationToken);
            break;
        case RideStatus.New:
        case RideStatus.ConfirmedByDriver:
        case RideStatus.Boarding:
        case RideStatus.Canceled:
        case RideStatus.Rejected:
            break;
        default:
            throw new InvalidEnumArgumentException(nameof(command.RideStatus), Enum.GetUnderlyingType(typeof(RideStatus)) == typeof(int) ? (int)command.RideStatus : 0, typeof(RideStatus)); // 🤷🏿‍♀️🤷🏿‍♀️🤷🏿‍♀️
    }
}
```

```csharp
public ValidateOptionsResult Validate(string? name, BuildTemplatesOptions options) => options switch
{
    _ when options?.WindowsServer2022 is null => ValidateOptionsResult.Fail("The Windows Server 2022 options are required."),
    _ when options?.WindowsServer2019 is null => ValidateOptionsResult.Fail("The Windows Server 2019 options are required."),
    _ when options?.WindowsServer2016 is null => ValidateOptionsResult.Fail("The Windows Server 2016 options are required."),
    _ when PropertiesNotMatch(options?.WindowsServer2022, _config!.WindowsServer2022) => ValidateOptionsResult.Fail("The Windows Server 2022 option properties do not match the base line."),
    _ when PropertiesNotMatch(options?.WindowsServer2019, _config!.WindowsServer2019) => ValidateOptionsResult.Fail("The Windows Server 2019 option properties do not match the base line."),
    _ when PropertiesNotMatch(options?.WindowsServer2016, _config!.WindowsServer2016) => ValidateOptionsResult.Fail("The Windows Server 2016 option properties do not match the base line."),
    _ when IsInvalidBuildTemplateOption(options?.WindowsServer2022) => ValidateOptionsResult.Fail("The Windows Server 2022 option properties are required."),
    _ when IsInvalidBuildTemplateOption(options?.WindowsServer2019) => ValidateOptionsResult.Fail("The Windows Server 2019 option properties are required."),
    _ when IsInvalidBuildTemplateOption(options?.WindowsServer2016) => ValidateOptionsResult.Fail("The Windows Server 2016 option properties are required."),
    _ => ValidateOptionsResult.Success,
};
```
