[Back to notes index](../README.md)

| [Previous: Dependency injection, options, and logging](06-dependency-injection-options-and-logging.md) | [Notes index](../README.md) | [Next: Controllers, REST, and OpenAPI](08-controllers-rest-and-openapi.md) |
| --- | --- | --- |

# 7. Minimal APIs

## When Minimal APIs fit

Minimal APIs define HTTP endpoints directly with route mapping methods. They are useful for small services, focused APIs, and applications where endpoint behavior is easy to see in one place. The word minimal describes the endpoint model, not the level of testing, validation, or structure an application should have.

A Minimal API can still use dependency injection, middleware, endpoint groups, filters, OpenAPI, and separate application services. Keep business rules out of a large Program.cs file by moving them into services as the app grows.

## A small reading-list API

This in-memory example supports listing, finding, and creating items:

```csharp
using Microsoft.AspNetCore.Http.HttpResults;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenApi();

var app = builder.Build();
var items = new List<ReadingItem>
{
    new(1, "HTTP fundamentals", false),
    new(2, "Dependency injection", true)
};

var routes = app.MapGroup("/api/reading-items")
    .WithTags("Reading items");

routes.MapGet("/", () => TypedResults.Ok(items));

routes.MapGet("/{id:int}", (int id) => FindItem(id, items));

routes.MapPost("/", (CreateReadingItem request) =>
{
    var item = new ReadingItem(
        items.Count == 0 ? 1 : items.Max(existing => existing.Id) + 1,
        request.Title,
        false);

    items.Add(item);
    return TypedResults.Created($"/api/reading-items/{item.Id}", item);
});

app.MapOpenApi();
app.Run();

static Results<Ok<ReadingItem>, NotFound> FindItem(
    int id,
    List<ReadingItem> items)
{
    var item = items.SingleOrDefault(existing => existing.Id == id);
    if (item is null)
    {
        return TypedResults.NotFound();
    }

    return TypedResults.Ok(item);
}

public sealed record ReadingItem(int Id, string Title, bool IsRead);
public sealed record CreateReadingItem(string Title);
```

The application stores data in memory, so changes disappear when it restarts. The sample demonstrates endpoint mapping and response types, not durable storage.

## Binding and dependency injection

Minimal APIs bind simple values from route and query parameters. Complex request objects are read from the request body. Registered services can be supplied as handler parameters. CancellationToken represents the request cancellation signal and should flow to database or network calls.

Use explicit types when they make the data source clear. A string parameter whose name matches a route value comes from the route. A search term such as query can come from the query string. A service registered in the container is provided through dependency injection.

## Responses and OpenAPI

Choose a status code that describes the result. A new resource commonly returns 201 Created and a location. A missing resource returns 404. A successful request that has no body can return 204. Typed results make possible response shapes easier for the compiler and generated API description to understand.

Add the OpenAPI package and call AddOpenApi and MapOpenApi to publish an API description document. OpenAPI describes the contract. A visual documentation page is a separate tool and is not automatically the same thing as the OpenAPI document.

## Practice check

Run the sample and request GET /api/reading-items, GET /api/reading-items/1, and POST /api/reading-items with a JSON body containing a title. Restart the app and observe why the collection resets. Replace the list with a database-backed service in the data chapter.

## Common mistakes

- Leaving all routes, persistence, validation, and business rules in one file.
- Returning 200 for every outcome.
- Assuming an in-memory collection is persistent or safe for concurrent writes.
- Accepting a request object without validating its values.
- Publishing development-only API details unintentionally.

## References

- [APIs overview](https://learn.microsoft.com/aspnet/core/fundamentals/apis?view=aspnetcore-10.0)
- [Minimal APIs overview](https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis/overview?view=aspnetcore-10.0)
- [OpenAPI support in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0)
