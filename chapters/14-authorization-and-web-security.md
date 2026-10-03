[Back to notes index](../README.md)

| [Previous: Authentication, Identity, and cookies](13-authentication-identity-and-cookies.md) | [Notes index](../README.md) | [Next: Error handling, Problem Details, and diagnostics](15-errors-problem-details-and-diagnostics.md) |
| --- | --- | --- |

# 14. Authorization and web security

## Authorization answers what

Authorization decides whether the current user may perform an action. A signed-in user is not automatically allowed to edit every reading item.

Use policies to express reusable requirements:

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("ReadingItemEditor", policy =>
        policy.RequireRole("Editor"));
```

A controller can require that policy with Authorize, and an endpoint group can use RequireAuthorization. AllowAnonymous should be deliberate and limited to endpoints that are meant to be public.

Roles are useful for broad categories. Resource-based authorization is needed when the rule depends on a specific record, such as whether the current user owns the item. Load the resource, then ask IAuthorizationService to evaluate the user and resource. Do not trust an owner ID supplied by the request.

## CORS

Cross-Origin Resource Sharing controls whether a browser permits a page from one origin to read a response from another origin. It is not authentication and does not stop non-browser clients from sending requests.

Configure the known front-end origin:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("Frontend", policy =>
        policy.WithOrigins("https://app.example.com")
            .AllowAnyHeader()
            .AllowAnyMethod());
});

var app = builder.Build();

app.UseRouting();
app.UseCors("Frontend");
app.UseAuthentication();
app.UseAuthorization();
```

Do not combine wildcard origins with credentials. Use the smallest list of origins, methods, and headers that the application needs.

## Common security boundaries

- Enforce HTTPS in production. Use HSTS for sites that are consistently served over HTTPS.
- Cookie-authenticated browser requests need protection from cross-site request forgery. Use antiforgery tokens for state-changing forms.
- Razor encodes values by default. Avoid rendering untrusted HTML as raw markup.
- Use parameterized database operations. Do not concatenate user input into SQL.
- Store secrets outside source control and restrict access to them.
- Limit upload size and validate file content on the server.
- Apply authorization to the server endpoint, even when the interface hides a button.
- Use rate limiting where an endpoint is vulnerable to costly or abusive traffic.

Security settings depend on how the application is hosted. Proxy headers, cookie domains, redirect URLs, and TLS termination must be configured for the real deployment path.

## Practice check

Create a policy for users allowed to edit items. Test the endpoint as an anonymous user, a signed-in user without the role, and a user with the role. Add an ownership rule and confirm that changing an item identifier cannot grant access to another user's record.

## Common mistakes

- Treating CORS as a replacement for authorization.
- Allowing any origin because a browser request failed.
- Checking ownership using a request field controlled by the caller.
- Returning hidden data and relying on the UI to conceal it.
- Disabling antiforgery protection on cookie-authenticated forms without an alternative.

## References

- [Authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/introduction?view=aspnetcore-10.0)
- [Resource-based authorization](https://learn.microsoft.com/aspnet/core/security/authorization/resourcebased?view=aspnetcore-10.0)
- [Enable Cross-Origin Requests](https://learn.microsoft.com/aspnet/core/security/cors?view=aspnetcore-10.0)
- [Prevent Cross-Site Request Forgery](https://learn.microsoft.com/aspnet/core/security/anti-request-forgery?view=aspnetcore-10.0)
