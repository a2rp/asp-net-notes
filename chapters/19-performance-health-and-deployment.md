[Back to notes index](../README.md)

| [Previous: Testing ASP.NET Core apps](18-testing-aspnet-core-apps.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
| --- | --- | --- |

# 19. Performance, health checks, and deployment

## Measure before changing code

Performance work starts with a measurement and a clear goal. Record response time, throughput, memory use, database query time, and error rates for representative traffic. Use logs, metrics, traces, and .NET diagnostic tools to find the bottleneck instead of guessing.

For request handlers:

- Use asynchronous APIs for I/O and pass CancellationToken through the call chain.
- Avoid blocking on Task.Result or Task.Wait.
- Select only the data the response needs.
- Page large collections and use indexes that match common filters.
- Set limits for request size and expensive operations.
- Avoid creating large temporary object graphs for every request.

Caching can improve response time, but it also creates an invalidation and privacy problem. In-memory cache belongs to one process. A distributed cache can be shared by several instances. Output caching stores endpoint responses and must be configured so user-specific content is not shared between users.

## Health checks

Health checks let a hosting platform or orchestrator ask whether the process is alive or ready for traffic:

```csharp
builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/health/live");
```

A liveness check should answer whether the process can keep running. A readiness check can include critical dependencies such as a database. Do not reveal connection strings, internal hostnames, or sensitive diagnostic details in an unauthenticated health response. Add dependency checks through the matching provider package when needed.

## Publish the app

Publish creates the files needed to deploy an application:

```bash
dotnet restore
dotnet test
dotnet publish -c Release -o ./publish
```

Choose framework-dependent deployment when the host provides the matching runtime. Choose self-contained deployment when the app must carry its runtime. Verify operating system, architecture, environment configuration, and hosting requirements before publishing.

## Reverse proxies and production settings

Kestrel can run behind a reverse proxy or a managed hosting service. Forwarded headers carry the original client and scheme information through trusted infrastructure. Configure forwarded-header handling only for known proxies or networks, and run it early enough for later HTTPS and redirect logic to see the correct values.

Production deployments should provide:

- Current supported runtime patches.
- Protected secrets and connection strings.
- Durable shared storage for databases and uploads.
- A shared protected Data Protection key ring when several instances read the same authentication cookies.
- Health and readiness checks.
- Centralized logs, metrics, and traces.
- A tested backup and rollback plan.

Use the hosting platform's deployment guidance for TLS, process management, scaling, and network rules. Local launch settings are for development and are not a production configuration source.

## Practice check

Publish the reading-list app in Release mode and run it from the publish directory. Configure a readiness check that verifies the database. Test behind a local reverse proxy and confirm generated redirects use the public HTTPS scheme.

## Common mistakes

- Optimizing before identifying a bottleneck.
- Using process memory as durable state in a scaled app.
- Sharing cached private responses between different users.
- Treating liveness as proof that all dependencies are ready.
- Trusting forwarded headers from arbitrary clients.
- Deploying without applying supported runtime patches.

## References

- [Performance best practices for ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/performance-best-practices?view=aspnetcore-10.0)
- [Health checks in ASP.NET Core](https://learn.microsoft.com/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-10.0)
- [Host and deploy ASP.NET Core](https://learn.microsoft.com/aspnet/core/host-and-deploy/?view=aspnetcore-10.0)
- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)
