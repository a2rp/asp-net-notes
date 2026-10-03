[Back to notes index](../README.md)

| [Previous: Hosting, environments, and configuration](03-hosting-environments-and-configuration.md) | [Notes index](../README.md) | [Next: Routing and endpoint design](05-routing-and-endpoints.md) |
| --- | --- | --- |

# 4. Middleware and the request pipeline

## What middleware does

Middleware is a component in the HTTP request pipeline. It receives an HttpContext and decides whether to:

- Do work before the next component.
- Call the next component.
- Do work after the next component returns.
- Stop the pipeline and produce a response.

A request moves through middleware in registration order. The response unwinds in reverse order. A component that does not call the next component short-circuits the request. This is useful for a response that is complete, but it can also accidentally prevent routing or authorization from running.

## Build a small pipeline

A middleware class can keep cross-cutting work out of endpoint handlers:

```csharp
public sealed class RequestLogMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLogMiddleware> _logger;

    public RequestLogMiddleware(
        RequestDelegate next,
        ILogger<RequestLogMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var started = Stopwatch.StartNew();
        await _next(context);
        started.Stop();

        _logger.LogInformation(
            "Handled {Method} {Path} with {StatusCode} in {ElapsedMs} ms",
            context.Request.Method,
            context.Request.Path,
            context.Response.StatusCode,
            started.ElapsedMilliseconds);
    }
}
```

Add the required namespace for Stopwatch, then register the middleware before endpoints:

```csharp
app.UseMiddleware<RequestLogMiddleware>();
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));
```

This sample logs a request after the endpoint has completed. ASP.NET Core already provides request logging options through hosting and logging integrations. A custom component is useful when a specific application rule is needed, not just to recreate built-in behavior.

## Order changes behavior

A common minimal hosting pipeline is:

```csharp
app.UseExceptionHandler();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors("Frontend");
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

Register exception handling near the start so it can catch failures from later components. Routing selects an endpoint. Authentication establishes the current user. Authorization checks whether that user may access the selected endpoint. CORS must be placed so the selected endpoint and preflight requests receive the intended policy.

WebApplication supplies some routing behavior automatically. Explicit calls are useful when order needs to be clear or a middleware depends on the selected endpoint.

## Use, Run, and Map

Use adds middleware that can call the next component. Run adds a terminal delegate. Map branches the pipeline based on a path prefix. These methods solve different problems:

- Use is for work around downstream processing.
- Run is for a final response that ends this branch.
- Map is for a path-based branch with its own components.

Prefer the built-in middleware for exception handling, authentication, static files, compression, and other standard needs.

## Common mistakes

- Putting authentication after authorization.
- Registering a terminal delegate before an endpoint that should run.
- Changing response headers after the response has started.
- Reading a request body in middleware without restoring its position for later components.
- Logging tokens, passwords, or other sensitive values.

## Practice check

Add a log before and after the next middleware call. Request an endpoint and note the order of the log entries. Then temporarily make a component return a response without calling the next delegate and observe that later endpoints do not run.

## References

- [ASP.NET Core middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware?view=aspnetcore-10.0)
- [Write custom ASP.NET Core middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/write?view=aspnetcore-10.0)
