[Back to notes index](../README.md)

| [Previous: Routing and endpoint design](05-routing-and-endpoints.md) | [Notes index](../README.md) | [Next: Minimal APIs](07-minimal-apis.md) |
| --- | --- | --- |

# 6. Dependency injection, options, and logging

## Dependency injection

Dependency injection lets a class receive the services it needs instead of constructing them internally. ASP.NET Core has a built-in service container. Register an abstraction and its implementation before building the app:

```csharp
builder.Services.AddScoped<IReadingItemService, ReadingItemService>();
```

A constructor can then receive the service:

```csharp
public sealed class ReadingItemService(IReadingItemStore store)
{
    public Task<IReadOnlyList<ReadingItem>> GetAllAsync(
        CancellationToken cancellationToken) =>
        store.GetAllAsync(cancellationToken);
}
```

Constructor injection makes dependencies visible and easier to replace in tests. Avoid calling the service provider manually throughout application code. That pattern hides dependencies and makes the object harder to reason about.

## Choose a lifetime

- Transient creates a new instance each time it is requested.
- Scoped creates one instance for a request scope. Entity Framework Core DbContext is normally scoped.
- Singleton creates one instance for the application's lifetime.

Do not inject a scoped service into a singleton. The singleton would keep a request-scoped object beyond its intended lifetime. A singleton that stores mutable state must be thread-safe. A stateless service can often use a longer lifetime, but scoped is a safe default for request-oriented application services.

## Bind and validate options

Related configuration belongs in a typed options object:

```csharp
public sealed class ReadingListOptions
{
    public const string SectionName = "ReadingList";

    public int PageSize { get; set; } = 20;
}
```

Register the object and validate its values when the app starts:

```csharp
builder.Services
    .AddOptions<ReadingListOptions>()
    .Bind(builder.Configuration.GetSection(ReadingListOptions.SectionName))
    .Validate(options => options.PageSize is > 0 and <= 100,
        "PageSize must be between 1 and 100.")
    .ValidateOnStart();
```

Options validation moves configuration mistakes to startup instead of letting them fail on the first request. Use IOptions<T> for values that remain fixed after startup. Use IOptionsSnapshot<T> for scoped access to updated values in web apps. Use IOptionsMonitor<T> when a singleton needs to observe changes.

## Structured logging

Inject ILogger<T> where a class needs to record events:

```csharp
app.MapGet("/api/reading-items", (
    ILogger<Program> logger,
    IReadingItemService service,
    CancellationToken cancellationToken) =>
{
    logger.LogInformation("Reading the current item collection");
    return service.GetAllAsync(cancellationToken);
});
```

Structured logging stores values as named fields. Prefer a template such as "Loaded {ItemCount} items" over building a string with concatenation. Do not log credentials, access tokens, session identifiers, or private request bodies.

## Practice check

Register a small reading-list service, inject it into an endpoint, and log one useful event. Add an invalid PageSize setting and confirm that validation fails during startup. Then correct the value and run the app again.

## References

- [Dependency injection in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-10.0)
- [Options pattern in .NET](https://learn.microsoft.com/dotnet/core/extensions/options)
- [Logging in .NET](https://learn.microsoft.com/dotnet/core/extensions/logging)
