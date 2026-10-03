[Back to notes index](../README.md)

| Previous: Index | [Notes index](../README.md) | [Next: .NET setup and project structure](02-dotnet-setup-and-project-structure.md) |
| --- | --- | --- |

# 1. ASP.NET Core and HTTP basics

## What ASP.NET Core does

ASP.NET Core is the web framework in .NET for receiving HTTP requests and producing HTTP responses. The web server accepts a connection, the framework builds an HttpContext, and the configured application pipeline decides what the request means.

A useful first model is:

1. A client sends a request with a method, path, headers, and optionally a body.
2. The server accepts the request and creates an HttpContext.
3. Middleware can inspect or change the request, call the next middleware, and inspect the response.
4. An endpoint handles the request and returns a result.
5. The response travels back through the pipeline to the client.

ASP.NET Core includes the server, routing, dependency injection, configuration, logging, and several ways to build web interfaces and APIs. Kestrel is the built-in cross-platform HTTP server. In production, a reverse proxy or a managed host may sit in front of it.

## HTTP request and response parts

A request has a method such as GET or POST, a path such as /api/reading-items, headers such as Accept, and sometimes a body. A response has a status code, headers, and optionally a body.

Common method intent:

- GET reads a representation and should not change server state.
- POST submits data or asks the server to create a resource.
- PUT replaces a resource at a known address.
- PATCH changes part of a resource.
- DELETE asks the server to remove a resource.

Common status codes include 200 OK for a successful read, 201 Created for a new resource, 204 No Content for success without a response body, 400 Bad Request for invalid input, 401 Unauthorized when authentication is missing, 403 Forbidden when access is denied, 404 Not Found when a resource does not exist, and 500 Internal Server Error for an unexpected server failure.

Do not trust values just because they came from a browser. Validate request data on the server and make authorization decisions on the server.

## The smallest useful app

The modern hosting model puts setup and endpoint mapping in Program.cs:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => Results.Text("Reading list service"));
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));

app.Run();
```

Run it from the project directory with dotnet run. The console prints a local address. A request to / receives a 200 response with plain text. The /health endpoint returns JSON because the result is an object.

A request can be checked from a terminal:

```bash
curl -i https://localhost:5001/health
```

Use the address printed by dotnet run. The port can differ between machines and projects.

## Where each concern belongs

Endpoints describe what a path does. Middleware handles cross-cutting request work such as exception handling, HTTPS redirection, authentication, and static files. Services hold reusable application behavior. Configuration supplies environment-specific settings. Keeping these responsibilities separate makes it easier to test and change the app.

ASP.NET Core does not require one UI style. Minimal APIs are concise endpoint definitions. Controllers organize HTTP APIs with attributes and action methods. Razor Pages and MVC render HTML on the server. Blazor builds interactive web interfaces with .NET.

## Common mistakes

- Returning a successful status for an operation that failed.
- Putting database work directly into every endpoint.
- Exposing exception details to public clients.
- Treating 401 and 403 as interchangeable.
- Assuming an HTTP request body is valid or safe.

## Practice check

Start a new web project, run it, and inspect the response headers with curl -i. Change /health to return a different status and JSON value. Notice that the method, path, status, and response body are separate parts of HTTP.

## References

- [ASP.NET Core overview](https://learn.microsoft.com/aspnet/core/overview?view=aspnetcore-10.0)
- [HTTP overview](https://learn.microsoft.com/aspnet/core/fundamentals/?view=aspnetcore-10.0)
- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)
