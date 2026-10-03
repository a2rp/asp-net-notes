[Back to notes index](../README.md)

| [Previous: ASP.NET Core and HTTP basics](01-aspnet-core-and-http.md) | [Notes index](../README.md) | [Next: Hosting, environments, and configuration](03-hosting-environments-and-configuration.md) |
| --- | --- | --- |

# 2. .NET setup and project structure

## SDK, runtime, and target framework

The .NET SDK is the development toolset. It includes the command line, compiler, templates, and build tools. The runtime executes applications. Install the SDK that matches the framework you plan to target, then check what the machine can use:

```bash
dotnet --info
dotnet --list-sdks
dotnet --list-runtimes
```

This notes collection uses .NET 10 and C#. The target framework is written in the project file as net10.0. Keep the SDK and runtime patched. The framework version does not pin a particular monthly patch.

## Create a project

For a small HTTP API based on Minimal APIs:

```bash
dotnet new webapi -n ReadingList.Api
cd ReadingList.Api
dotnet run
```

The default Web API template uses Minimal APIs. To start with controllers, pass the controller option:

```bash
dotnet new webapi --use-controllers -n ReadingList.Api
```

For server-rendered pages, the built-in templates include Razor Pages and Blazor:

```bash
dotnet new webapp -n ReadingList.Web
dotnet new blazor -n ReadingList.Blazor
```

Templates can change between SDK releases. Use dotnet new list to inspect the templates installed with the SDK.

## Read the project files

A typical web project contains:

- Program.cs, where the host, services, middleware, and endpoints are configured.
- A project file ending in .csproj, which declares the target framework and package references.
- appsettings.json, which stores non-secret defaults.
- Properties/launchSettings.json, which stores local development launch profiles.
- wwwroot, for public static web assets when the application serves them.
- bin and obj, generated build output that should not be committed.

A minimal project file looks like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

Nullable reference types help the compiler identify places where a value may be missing. Treat the warnings as useful questions about the data flow.

## Useful CLI commands

```bash
dotnet restore
dotnet build
dotnet run
dotnet watch
dotnet test
```

Run commands from the solution or project directory. Use dotnet run --project path/to/ReadingList.Api when the current directory is elsewhere. dotnet watch rebuilds and restarts the app as source files change.

For a multi-project solution, create a solution file, add projects, and reference shared code explicitly:

```bash
dotnet new sln -n ReadingList
dotnet sln ReadingList.sln add src/ReadingList.Api/ReadingList.Api.csproj
dotnet sln ReadingList.sln add tests/ReadingList.Api.Tests/ReadingList.Api.Tests.csproj
```

A project reference is not the same as a NuGet package reference. Use project references for code in the same solution and package references for shared libraries distributed through NuGet.

## Practice check

Create the API template, run it, and inspect its project file. Build it before changing anything. Then add a simple endpoint, run again, and use the console address to request it. This separates SDK or restore problems from application code problems.

## Common mistakes

- Installing only a runtime when you need to compile.
- Editing files under bin or obj instead of source files.
- Committing local secrets or generated output.
- Adding a package without checking whether the framework already includes the feature.
- Assuming every template creates the same files across SDK versions.

## References

- [Install .NET](https://learn.microsoft.com/dotnet/core/install/)
- [The .NET CLI](https://learn.microsoft.com/dotnet/core/tools/)
- [ASP.NET Core project templates](https://learn.microsoft.com/aspnet/core/fundamentals/?view=aspnetcore-10.0)
