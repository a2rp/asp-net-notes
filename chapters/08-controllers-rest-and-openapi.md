[Back to notes index](../README.md)

| [Previous: Minimal APIs](07-minimal-apis.md) | [Notes index](../README.md) | [Next: Model binding, validation, and filters](09-model-binding-validation-and-filters.md) |
| --- | --- | --- |

# 8. Controllers, REST, and OpenAPI

## Controller responsibilities

Controllers group related HTTP actions into classes. They fit APIs that benefit from a clear class boundary, attributes, action filters, and familiar MVC conventions. Register controllers and map them in Program.cs:

```csharp
builder.Services.AddControllers();
builder.Services.AddOpenApi();

var app = builder.Build();

app.MapOpenApi();
app.MapControllers();
```

The controller should translate HTTP input into an application service call and translate the result into an HTTP response. Keep business rules in services so they can be tested without constructing an HTTP request.

## A controller for reading items

This example assumes the application service returns null when an item does not exist:

```csharp
[ApiController]
[Route("api/reading-items")]
public sealed class ReadingItemsController(
    IReadingItemService service) : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<IReadOnlyList<ReadingItem>>> GetAll(
        CancellationToken cancellationToken)
    {
        var items = await service.GetAllAsync(cancellationToken);
        return Ok(items);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<ReadingItem>> GetById(
        int id,
        CancellationToken cancellationToken)
    {
        var item = await service.GetByIdAsync(id, cancellationToken);
        if (item is null)
        {
            return NotFound();
        }

        return Ok(item);
    }

    [HttpPost]
    public async Task<ActionResult<ReadingItem>> Create(
        CreateReadingItemRequest request,
        CancellationToken cancellationToken)
    {
        var item = await service.CreateAsync(
            request.Title,
            cancellationToken);

        return CreatedAtAction(
            nameof(GetById),
            new { id = item.Id },
            item);
    }
}
```

A request DTO keeps the public contract separate from an EF Core entity:

```csharp
public sealed record CreateReadingItemRequest(
    [Required, StringLength(120)] string Title);
```

Add the DataAnnotations namespace for Required and StringLength. With ApiController, invalid model state normally produces a 400 response before the action runs.

## HTTP contract choices

Use a consistent resource path and return meaningful status codes:

- GET collection returns 200 and a collection, including an empty collection when there are no items.
- GET one item returns 200 or 404.
- POST creates an item and commonly returns 201 with a location.
- PUT replaces a resource and commonly returns 200 or 204.
- DELETE commonly returns 204 after a successful removal.

A controller does not make an API RESTful by itself. A clear resource model, appropriate methods, status codes, and stable representations matter more than the class style.

## OpenAPI

OpenAPI is a machine-readable description of routes, parameters, request bodies, and response shapes. It helps client generation and API review. Keep response contracts accurate and avoid exposing internal database models as public response types. The OpenAPI document is separate from a visual API browser, which may require an additional tool.

## Practice check

Register a controller, request the collection, request an unknown identifier, and submit an empty title. Check the status code and response body for each case. Add a successful POST and confirm that the returned location points to the new item.

## Common mistakes

- Returning EF Core entities directly from every endpoint.
- Putting persistence and business rules inside controller actions.
- Returning 200 when the resource was not found.
- Accepting an entity as the write request and allowing clients to set server-owned fields.
- Documenting response codes that the action never returns.

## References

- [Controllers in ASP.NET Core Web API](https://learn.microsoft.com/aspnet/core/web-api/?view=aspnetcore-10.0)
- [Create web APIs with controllers](https://learn.microsoft.com/aspnet/core/web-api/?view=aspnetcore-10.0)
- [OpenAPI support in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0)
