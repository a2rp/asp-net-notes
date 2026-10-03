[Back to notes index](../README.md)

| [Previous: .NET setup and project structure](02-dotnet-setup-and-project-structure.md) | [Notes index](../README.md) | [Next: Middleware and the request pipeline](04-middleware-and-request-pipeline.md) |
| --- | --- | --- |

# 3. Hosting, environments, and configuration

## The host

The host starts the application, loads configuration, builds the dependency injection container, configures logging, and manages the server lifetime. In modern ASP.NET Core, WebApplication.CreateBuilder provides common defaults:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services and configure the app here.

var app = builder.Build();

// Configure middleware and map endpoints here.

app.Run();
```

The app can read the environment through builder.Environment. Common environment names are Development, Staging, and Production. These are names, not separate framework modes. The environment affects which configuration files and conditional setup are used.

## Configuration sources and precedence

ASP.NET Core combines configuration providers into one configuration view. Common sources include appsettings.json, appsettings.{Environment}.json, development user secrets, environment variables, and command-line arguments. Later providers override earlier providers for matching keys.

A JSON file can contain non-secret defaults:

```json
{
  "ReadingList": {
    "PageSize": 20,
    "AllowGuestReads": true
  }
}
```

Configuration keys use a colon between levels. Environment variables use a double underscore in place of the colon, so ReadingList__PageSize can override ReadingList:PageSize. This is useful in containers and deployment platforms.

Read a value directly when it is a small, one-off setting:

```csharp
var pageSize = builder.Configuration.GetValue<int>("ReadingList:PageSize", 20);
```

For a related group of values, use the options pattern. The next chapter explains registering and validating options in more detail.

## Environment-specific behavior

Use development-only behavior only in development:

```csharp
if (builder.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

The default web host often enables useful development behavior for the Development environment. Do not expose detailed exception pages in production. They can reveal file paths, stack traces, and internal values.

Keep a safe baseline in appsettings.json and place environment overrides in appsettings.Development.json or appsettings.Production.json. Environment-specific files are useful for public settings such as a feature flag or endpoint address. They are not a safe place for passwords or API keys in a public repository.

## Local secrets

For local development, initialize and use the user secrets store from the project directory:

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:ReadingList" "replace-with-a-local-value"
dotnet user-secrets list
```

User secrets keep values out of the project tree, but they are not an encrypted production vault. Use a managed secret store or the hosting platform's protected configuration for deployed applications. Do not paste real credentials into notes, terminal history, issue reports, or Git.

## Practice check

Set ReadingList:PageSize in appsettings.json, then override it with an environment variable named ReadingList__PageSize. Log or return the effective value in a local-only endpoint. Remove that endpoint before deploying if the value should not be public.

## Common mistakes

- Assuming configuration files are loaded in alphabetical order.
- Checking the wrong environment name because environment variables differ by host.
- Storing credentials in appsettings.json.
- Treating user secrets as a production secret manager.
- Relying on a missing value to silently become a valid default.

## References

- [Configuration in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/?view=aspnetcore-10.0)
- [Use multiple environments in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/environments?view=aspnetcore-10.0)
- [Safe storage of app secrets in development](https://learn.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-10.0)
