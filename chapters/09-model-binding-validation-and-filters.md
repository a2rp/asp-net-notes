[Back to notes index](../README.md)

| [Previous: Controllers, REST, and OpenAPI](08-controllers-rest-and-openapi.md) | [Notes index](../README.md) | [Next: Razor Pages and MVC](10-razor-pages-and-mvc.md) |
| --- | --- | --- |

# 9. Model binding, validation, and filters

## Model binding sources

Model binding turns request data into method parameters or objects. In controller actions, common sources are:

- Route values for a resource identifier.
- Query values for search, filtering, and paging.
- Request bodies for JSON or another configured format.
- Form fields for browser form submissions.
- Headers for values that belong in a header.
- Services from dependency injection.

Make the source clear when a method has values with similar names:

```csharp
[HttpGet("search")]
public IActionResult Search(
    [FromQuery] string? term,
    [FromQuery] int page = 1)
{
    if (page < 1)
    {
        return BadRequest("Page must be positive.");
    }

    return Ok(new { term, page });
}
```

A request body is usually read once. Avoid trying to bind the same body to multiple parameters.

## Validate the shape of input

Use a request DTO with validation rules:

```csharp
public sealed class CreateReadingItemRequest
{
    [Required]
    [StringLength(120, MinimumLength = 2)]
    public string Title { get; set; } = string.Empty;
}
```

Add the System.ComponentModel.DataAnnotations namespace. An API controller marked with ApiController automatically returns a 400 response when model validation fails. For MVC views, invalid model state should be shown back to the user with useful field messages.

Validation has layers:

1. Binding checks whether request data can be converted to the target type.
2. DTO validation checks whether the shape and basic values are acceptable.
3. Application rules check whether the operation makes sense in the current state.
4. Authorization checks whether this user may perform the operation.

A DTO attribute cannot prove that a title is unique in the database. That rule belongs in application logic and must handle concurrent requests safely.

## Avoid over-posting

Do not bind a database entity directly from untrusted input when the entity contains fields the client must not control. A dedicated request type makes the accepted fields explicit. Assign server-owned values such as owner ID, creation time, and permission flags in trusted server code.

Return a separate response type when the client should see only part of the stored record. That keeps internal columns out of the public contract.

## Filters and middleware

Filters run at specific points around MVC actions or endpoint execution:

- Authorization filters run early and are for authorization decisions.
- Resource filters wrap much of MVC processing.
- Action filters run before and after a controller action.
- Exception filters handle some MVC action exceptions.
- Result filters wrap result execution.
- Endpoint filters wrap Minimal API handlers and can inspect bound arguments.

Middleware covers the whole request pipeline and works across endpoint types. A filter is more focused on MVC or a specific endpoint group. Prefer the narrowest layer that matches the behavior.

## Practice check

Add a search action with a query term and page value. Send valid input, a page of zero, and a malformed page. Add a request DTO with title length rules and test empty, too short, valid, and too long titles.

## Common mistakes

- Treating client-side validation as a security boundary.
- Using one database entity as both request and response.
- Returning framework exception text to explain invalid input.
- Putting authorization decisions in a model validator.
- Adding a filter when middleware or a service is the simpler fit.

## References

- [Model binding in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/model-binding?view=aspnetcore-10.0)
- [Model validation in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/validation?view=aspnetcore-10.0)
- [Filters in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/controllers/filters?view=aspnetcore-10.0)
