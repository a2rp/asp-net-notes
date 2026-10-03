[Back to notes index](../README.md)

| [Previous: SignalR and gRPC](17-signalr-and-grpc.md) | [Notes index](../README.md) | [Next: Performance, health checks, and deployment](19-performance-health-and-deployment.md) |
| --- | --- | --- |

# 18. Testing ASP.NET Core apps

## Test at the right level

A unit test checks one class or rule without starting the web host. It is fast and helps explain the expected behavior of a service. An integration test starts the application in a test host and exercises routing, middleware, binding, filters, and serialization together.

Use integration tests for behavior that depends on the HTTP pipeline. Avoid duplicating every small assertion at both levels. Keep test data independent so a test does not depend on execution order.

## Create a test host

Add the Microsoft.AspNetCore.Mvc.Testing package to the test project. For top-level Program.cs, expose the generated Program type to the test assembly:

```csharp
public partial class Program { }
```

A test can use WebApplicationFactory to make an in-process client:

```csharp
public sealed class HealthEndpointTests(
    WebApplicationFactory<Program> factory)
    : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client = factory.CreateClient();

    [Fact]
    public async Task Health_returns_success()
    {
        var response = await _client.GetAsync("/health");

        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    }
}
```

Add the xUnit and MVC testing namespaces to the test file. This test sends an HTTP request through the app rather than calling an endpoint delegate directly.

For richer tests, derive from WebApplicationFactory and configure test services. Replace a production service with a fake, or provide a test database. Ensure the database provider behaves like the production provider for the behavior under test. EF Core's in-memory provider does not behave like a relational database.

## Test important outcomes

Test successful requests and important failures:

- The expected status code and response shape.
- Invalid input and validation responses.
- Missing records and authorization denials.
- Authentication requirements.
- Database behavior that matters to users.
- Error responses without internal exception data.

Use representative inputs. A test name should say what behavior it verifies. Avoid tests that only repeat a private implementation detail.

## Run tests

```bash
dotnet test
dotnet test --filter FullyQualifiedName~HealthEndpointTests
```

Run the full suite before pushing a change that affects shared behavior. If tests use external services, make that dependency explicit and provide a local test configuration or a documented test environment.

## Common mistakes

- Making an integration test call a production database.
- Sharing mutable test data between parallel tests.
- Checking only that a response is 200 without checking the important body.
- Using the in-memory EF provider to make claims about SQL behavior.
- Bypassing middleware in a test intended to verify the full request pipeline.

## Practice check

Add a test for the reading-list collection and a test for a missing item. Add a validation test for an empty title. Then replace the database service for tests and confirm no test needs an external production account.

## References

- [Integration tests in ASP.NET Core](https://learn.microsoft.com/aspnet/core/test/integration-tests?view=aspnetcore-10.0)
- [Unit testing in .NET](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)
- [Testing EF Core applications](https://learn.microsoft.com/ef/core/testing/)
