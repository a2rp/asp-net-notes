[Back to notes index](../README.md)

| [Previous: Authorization and web security](14-authorization-and-web-security.md) | [Notes index](../README.md) | [Next: Static files, uploads, and downloads](16-static-files-uploads-and-downloads.md) |
| --- | --- | --- |

# 15. Error handling, Problem Details, and diagnostics

## Handle failures centrally

An unexpected exception should not become a successful response or expose internal details. Register Problem Details and exception handling near the start of the pipeline:

```csharp
builder.Services.AddProblemDetails();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler();
    app.UseHsts();
}

app.UseStatusCodePages();
```

Development exception pages help diagnose local failures. Production responses should be safe and consistent. Log the exception on the server with a request or trace identifier, then return a response that does not reveal stack traces, file paths, or secret values.

## Problem Details

Problem Details is a standard JSON shape for describing an HTTP error. It can include a status, title, detail, and instance. Keep the detail suitable for the caller. Do not copy exception messages into a public response.

An endpoint can return a deliberate problem response:

```csharp
app.MapGet("/api/reading-items/{id:int}", (int id) =>
{
    if (id < 1)
    {
        return TypedResults.Problem(
            title: "Invalid reading item identifier",
            statusCode: StatusCodes.Status400BadRequest);
    }

    return TypedResults.Ok(new { Id = id, Title = "HTTP fundamentals" });
});
```

For an API, use consistent status codes and stable error fields so clients can react without parsing human prose. Validation errors should identify fields and explain how a request can be corrected.

## Logs and traces

Use structured logging with useful event names and values. Log enough context to understand the failure, but avoid passwords, tokens, session cookies, and private request bodies. Set levels according to importance:

- Trace and Debug are for detailed development diagnostics.
- Information records normal application events.
- Warning records an unexpected condition that the app handled.
- Error records an operation that failed.
- Critical records a failure that may require immediate attention.

Distributed traces connect work across components. ASP.NET Core uses diagnostic activities, and OpenTelemetry can export traces and metrics to an observability platform. Use a trace identifier to connect an API response with the corresponding server logs.

## Distinguish expected outcomes from faults

A missing record is usually a 404 result, not an exception. Invalid input is usually a 400 response with validation details. An unexpected database outage is an error that should be logged and returned as a safe 5xx response. Use exceptions for exceptional failures, not ordinary branching.

## Practice check

Make one endpoint return a deliberate validation problem and another throw an exception locally. Compare the client response and the server logs in Development and Production. Confirm that the production response contains no stack trace.

## Common mistakes

- Returning status 200 with an error message in the body.
- Catching every exception and silently returning an empty value.
- Returning exception.ToString() to an unauthenticated client.
- Logging the same exception at multiple layers with no added context.
- Treating a health probe failure as the same thing as an application exception.

## References

- [Handle errors in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/error-handling?view=aspnetcore-10.0)
- [Problem Details in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/error-handling-api?view=aspnetcore-10.0)
- [Logging overview](https://learn.microsoft.com/dotnet/core/extensions/logging)
- [Distributed tracing in .NET](https://learn.microsoft.com/dotnet/core/diagnostics/distributed-tracing)
