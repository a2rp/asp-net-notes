[Back to notes index](../README.md)

| [Previous: Data access with Entity Framework Core](12-data-access-with-ef-core.md) | [Notes index](../README.md) | [Next: Authorization and web security](14-authorization-and-web-security.md) |
| --- | --- | --- |

# 13. Authentication, Identity, and cookies

## Authentication answers who

Authentication establishes the identity associated with a request. Authorization, covered next, decides whether that identity may perform an operation.

ASP.NET Core Identity manages user accounts, password hashing, sign-in, roles, tokens, and related account workflows. Use the supported account system rather than writing a password store from scratch. Identity normally stores account data through Entity Framework Core, so the app needs an application user context and a database provider.

A typical service setup looks like this:

```csharp
builder.Services
    .AddDefaultIdentity<IdentityUser>(options =>
    {
        options.SignIn.RequireConfirmedAccount = true;
        options.Password.RequiredLength = 12;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

Templates and package choices differ between Razor Pages, MVC, and API projects. Apply account UI and database setup from the matching ASP.NET Core Identity documentation.

## Cookie authentication

Browser applications commonly use a protected authentication cookie after sign-in. The browser sends the cookie with later requests. The server validates it and reconstructs the user's identity. Cookie settings should require HTTPS, use an appropriate SameSite policy, and have a deliberate expiration and renewal policy.

Authentication middleware must run before authorization middleware:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

Map protected endpoints after the middleware is registered. Cookie authentication uses ASP.NET Core Data Protection to protect ticket data. In a multi-instance deployment, configure a shared and protected key ring so another instance can read the cookie.

Sign-out changes server authentication state and should use a protected form or another anti-forgery mechanism. Do not use a state-changing GET request for sign-out.

## Tokens for APIs

A browser cookie and an access token solve different client scenarios. An API called by a separate client may accept a bearer token issued by a trusted identity provider. Validate issuer, audience, signature, and lifetime with supported authentication middleware. Do not invent a token format or store long-lived bearer tokens in browser-accessible storage without understanding the exposure.

Authentication proves the caller's identity. It does not automatically grant access to every resource. Add authorization rules to decide what that identity can do.

## Practice check

Add the framework identity services to a local app, configure a development database, and exercise registration, confirmation, sign-in, and sign-out. Inspect protected pages while signed in and signed out. Never use a real personal password or production account in a local experiment.

## Common mistakes

- Storing passwords as plain text or with a custom reversible encoding.
- Assuming that a signed-in user is authorized for every record.
- Placing signing keys or token secrets in source control.
- Disabling cookie security settings to make local testing easier.
- Returning authentication tokens in URLs or logs.

## References

- [Introduction to ASP.NET Core Identity](https://learn.microsoft.com/aspnet/core/security/authentication/identity?view=aspnetcore-10.0)
- [Configure cookie authentication](https://learn.microsoft.com/aspnet/core/security/authentication/cookie?view=aspnetcore-10.0)
- [Authentication in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authentication/?view=aspnetcore-10.0)
