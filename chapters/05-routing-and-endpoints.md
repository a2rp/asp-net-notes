[Back to notes index](../README.md)

| [Previous: Middleware and the request pipeline](04-middleware-and-request-pipeline.md) | [Notes index](../README.md) | [Next: Dependency injection, options, and logging](06-dependency-injection-options-and-logging.md) |
| --- | --- | --- |

# 5. Routing and endpoint design

## Paths, route values, and query strings

Routing matches an incoming request to an endpoint. A route template can include literal segments and named parameters:

```csharp
app.MapGet("/reading-items/{id:int}", (int id) =>
    Results.Ok(new { Id = id, Title = "Distributed systems notes" }));
```

A request to /reading-items/42 binds 42 to id. The int constraint requires that route segment to parse as an integer. A route constraint helps select a route. It is not a replacement for validation or an authorization check.

Use route values to identify a resource, such as /reading-items/42. Use query values for optional filters or pagination, such as /reading-items?isRead=true&page=2. Keep resource routes predictable and use HTTP methods to describe the operation.

## Define an endpoint group

MapGroup shares a prefix and endpoint metadata:

```csharp
var readingItems = app.MapGroup("/api/reading-items")
    .WithTags("Reading items");

readingItems.MapGet("/", () =>
    Results.Ok(new[] { "Distributed systems notes", "HTTP notes" }));

readingItems.MapGet("/{id:int}", (int id) =>
    Results.Ok(new { Id = id, Title = "Distributed systems notes" }));
```

A group is useful for applying shared authorization, filters, tags, or a common prefix. Keep endpoints separated by resource and behavior. A single endpoint should not silently perform several unrelated operations.

## Route matching details

Route templates can include optional parameters, defaults, constraints, and catch-all segments. Use these features only when they make the URL easier to understand. Avoid overlapping templates that differ only in subtle details. Clear route shapes make APIs easier to use and test.

Route values are strings until the framework binds them to parameter types. If a value cannot be bound or does not satisfy the route constraint, the endpoint may not match. Query values can be missing, repeated, or malformed, so validate values that affect database queries or resource access.

Name routes when another part of the app needs a stable way to generate a link:

```csharp
app.MapGet("/api/reading-items/{id:int}", (int id) =>
        Results.Ok(new { Id = id }))
    .WithName("GetReadingItem");
```

A route name is application metadata. It does not change the URL by itself.

## Endpoint design habits

- Use nouns for resource paths and HTTP methods for actions.
- Return a resource location after creating a resource when that location is known.
- Decide what a missing resource means and use a consistent status code.
- Keep URLs independent from internal database table names.
- Avoid putting secrets or private data in URLs because URLs can appear in logs and browser history.
- Treat route constraints as matching rules, not as security boundaries.

## Practice check

Add a route for one item and a route for the collection. Try an integer identifier, a non-integer identifier, and an identifier that does not exist. Compare whether the endpoint matched with what the application should return for a missing record.

## References

- [Routing in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/routing?view=aspnetcore-10.0)
- [Route groups in Minimal APIs](https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-10.0)
