[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | Next: End |
| --- | --- | --- |

# 99. Complete questions and answers

These questions review the core ideas from the chapters. Use them to check whether you can explain the reason behind a framework feature, not only remember its name.

## 1. ASP.NET Core and HTTP basics

**1. What is ASP.NET Core?**  
ASP.NET Core is the web framework in .NET for handling HTTP requests and building APIs, server-rendered pages, and interactive web apps.

**2. What is the difference between a request and a response?**  
A request comes from a client and includes a method, path, headers, and sometimes a body. The server returns a response with a status code, headers, and an optional body.

**3. What does middleware do?**  
Middleware runs in the request pipeline. It can inspect a request, call the next component, change the response, or stop processing with its own response.

**4. When should an API return 401, 403, and 404?**  
Use 401 when authentication is missing or invalid, 403 when the caller is authenticated but not allowed, and 404 when the requested resource does not exist or should not be disclosed.

## 2. .NET setup and project structure

**5. What is the difference between the .NET SDK and runtime?**  
The SDK includes the compiler and build tools. The runtime executes compiled applications.

**6. Which template should start a small API?**  
The webapi template is a good starting point. It uses Minimal APIs by default in current SDKs. The controller option creates a controller-based API shape.

**7. What does the target framework in a project file control?**  
It identifies the .NET API surface and runtime target used to build the project. It does not pin a specific monthly security patch.

**8. Why should bin and obj normally stay out of Git?**  
They contain generated build and restore output. The SDK recreates them, so source control should keep the project inputs instead.

## 3. Hosting, environments, and configuration

**9. What does WebApplication.CreateBuilder provide?**  
It creates the host builder with common configuration, logging, dependency injection, and web hosting defaults.

**10. What is the purpose of Development and Production environments?**  
They let the app choose safe environment-specific behavior and settings. Development can show detailed diagnostics, while Production should return safe errors.

**11. How do configuration providers override each other?**  
Providers are added in an order. A later provider generally wins when it supplies the same key, so environment variables and command-line values can override JSON defaults.

**12. Are user secrets suitable for production credentials?**  
No. They are a local development convenience that keeps values out of the project tree. Production secrets belong in a protected secret store or hosting configuration.

## 4. Middleware and the request pipeline

**13. Why does middleware order matter?**  
Each component sees requests in registration order and responses on the way back out. A component placed too late may not see failures or may prevent later processing.

**14. What is the difference between Use, Run, and Map?**  
Use adds middleware that can call the next component. Run adds a terminal delegate. Map branches processing based on a path prefix.

**15. Where should exception handling usually be registered?**  
Near the start of the pipeline so it can handle exceptions from downstream middleware and endpoints.

**16. Why should response headers be set before the response starts?**  
Once the server has sent response headers or body bytes, those headers cannot be changed reliably.

## 5. Routing and endpoint design

**17. When should an identifier be in the route instead of the query string?**  
A path segment commonly identifies one resource. Query values are useful for optional filters, sorting, and paging.

**18. Does an integer route constraint validate all business rules?**  
No. It checks whether the route segment matches the expected shape. The app still checks ranges, existence, ownership, and other rules.

**19. What is a route group useful for?**  
It shares a path prefix and endpoint metadata such as tags, authorization, and filters across related endpoints.

**20. Should database table names determine public URLs?**  
Usually not. Public routes should describe the resource contract and remain stable even if the database schema changes.

## 6. Dependency injection, options, and logging

**21. Why use dependency injection?**  
It makes a class's dependencies visible and lets the application provide the appropriate implementation and lifetime.

**22. When should a service be scoped?**  
Use a scoped lifetime for work that belongs to one request, including the usual Entity Framework Core DbContext lifetime.

**23. Why is injecting a scoped service into a singleton unsafe?**  
The singleton can retain a request-scoped object beyond the request that created it. This causes incorrect lifetime behavior and can expose request data across users.

**24. Why validate options at startup?**  
It makes invalid configuration fail early with a useful message instead of failing later during a request.

## 7. Minimal APIs

**25. What are Minimal APIs?**  
They are route-mapped HTTP handlers with direct access to request values and registered services. They can still use validation, filters, authorization, OpenAPI, and application services.

**26. When are Minimal APIs a good fit?**  
They work well for focused APIs and smaller services where concise endpoint mapping keeps the behavior clear.

**27. Why use typed results?**  
Typed results make response shapes more explicit and help tools describe the endpoint contract.

**28. Why is an in-memory list unsuitable for durable application data?**  
It disappears when the process restarts, is local to one instance, and needs careful concurrency handling.

## 8. Controllers, REST, and OpenAPI

**29. What does ApiController add?**  
It applies API conventions such as automatic validation responses and clearer binding behavior for controller actions.

**30. Why use request DTOs instead of binding database entities?**  
A DTO defines exactly which fields the client may submit and avoids exposing internal persistence fields.

**31. What does CreatedAtAction communicate?**  
It returns a successful creation response with a location that points to the new resource.

**32. What is OpenAPI?**  
It is a machine-readable description of API routes, inputs, and responses. A visual API browser is a separate presentation tool.

## 9. Model binding, validation, and filters

**33. What does model binding do?**  
It converts request route values, query values, form values, headers, or body data into method parameters and objects.

**34. Is client-side validation enough?**  
No. Clients can bypass browser code, so the server validates every input before using it.

**35. What should basic DTO validation check?**  
It can check required values, length, format, and simple ranges. Rules that depend on database state belong in application logic.

**36. When is a filter a better fit than middleware?**  
A filter is useful for behavior tied to MVC actions or endpoint execution. Middleware is broader and applies across the request pipeline.

## 10. Razor Pages and MVC

**37. How do Razor Pages and MVC differ?**  
Razor Pages organizes handlers around pages. MVC organizes request handling around controllers and views. Both use the same ASP.NET Core host and services.

**38. What is the post-redirect-get pattern?**  
After a successful form post, the server redirects to a GET page. Refreshing that page will not resubmit the original form.

**39. Why should a form show server validation messages?**  
The server is the trusted validation boundary, and the user needs clear feedback when submitted data is rejected.

**40. Is Razor syntax the same as Blazor?**  
No. Razor is a syntax used by several ASP.NET Core technologies. Razor Pages and MVC render HTML on the server, while Blazor uses components and render modes.

## 11. Blazor Web Apps

**41. What is a Blazor component?**  
It is a reusable UI unit that combines markup, parameters, state, and event handling.

**42. What is a render mode?**  
It selects how a component is rendered and whether it becomes interactive on the server, in WebAssembly, or through another supported mode.

**43. Where does Interactive Server state live?**  
It lives in a server-side circuit while that connection is active. Important state must be saved in durable application storage.

**44. Can WebAssembly code contain a server secret?**  
No. Browser-delivered code and values can be inspected by the user. Keep trusted credentials and authorization decisions on the server.

## 12. Data access with Entity Framework Core

**45. What does DbContext represent?**  
It represents a unit of work that tracks entity changes and coordinates queries and saves for a database session.

**46. Why is DbContext usually scoped in a web app?**  
A request can use one context for its data work without sharing mutable tracking state across concurrent requests.

**47. When should AsNoTracking be used?**  
Use it for read-only queries where the returned entities will not be updated through that context.

**48. What is an EF Core migration?**  
It is a versioned description of a database schema change. Review it and plan how it will be applied in each environment.

## 13. Authentication, Identity, and cookies

**49. What does authentication answer?**  
It establishes who is making a request.

**50. What does ASP.NET Core Identity provide?**  
It provides account management features such as user storage, password hashing, sign-in, roles, and account tokens.

**51. What does a protected authentication cookie contain?**  
It carries protected authentication ticket data that the server uses to restore the user's identity on later requests.

**52. When are bearer tokens common?**  
They are common for APIs called by separate clients that present tokens issued by a trusted identity provider.

## 14. Authorization and web security

**53. What does authorization answer?**  
It decides whether an identity may perform an operation or access a resource.

**54. Is CORS an access control system for APIs?**  
No. CORS controls what browser scripts may read across origins. The API still needs authentication and authorization.

**55. What is resource-based authorization?**  
It checks access using the specific object, such as whether the current user owns a particular reading item.

**56. Why do cookie-authenticated forms need antiforgery protection?**  
Browsers send cookies automatically. An attacker may try to make a signed-in browser submit an unwanted state-changing request.

## 15. Error handling, Problem Details, and diagnostics

**57. What should a production error response avoid?**  
It should avoid stack traces, file paths, secret values, and internal implementation details.

**58. What is Problem Details used for?**  
It provides a consistent response structure for HTTP errors so clients can inspect a status, title, and other safe details.

**59. Should a missing database record be thrown as an exception?**  
Usually not. A normal missing-resource outcome should be represented as a 404 response.

**60. What makes a log structured?**  
It records named values separately from the message template, which makes the event easier to search and aggregate.

## 16. Static files, uploads, and downloads

**61. What belongs under wwwroot?**  
Public files that the app may serve directly, such as public stylesheets, scripts, and images.

**62. Why generate a server-side upload file name?**  
It avoids trusting user-controlled path text and reduces collisions and path traversal risk.

**63. Does a file extension prove a file is safe?**  
No. The extension and content type are supplied or influenced by the client. Validate content and apply size and storage rules.

**64. How should private downloads be served?**  
Look up the file by an application identifier, check the user's access to its record, and return a server-selected storage path.

## 17. SignalR and gRPC

**65. When is SignalR useful?**  
It is useful when connected clients need server push, live updates, or two-way messages.

**66. What is a SignalR group?**  
It is a set of connections that can receive the same server message. Membership must be authorized, not inferred from a group name.

**67. What does gRPC use for service contracts?**  
It commonly uses Protocol Buffers definitions to generate typed client and server code.

**68. Which should a browser use, SignalR or gRPC?**  
Choose based on the interaction. SignalR supports live messages; browser gRPC may need gRPC-Web or an HTTP API depending on the environment.

## 18. Testing ASP.NET Core apps

**69. What is the difference between a unit test and an integration test?**  
A unit test isolates one rule or class. An integration test runs several components together, often through the HTTP pipeline.

**70. What does WebApplicationFactory provide?**  
It starts an ASP.NET Core app in a test host and provides an HttpClient for in-process HTTP requests.

**71. Why is EF Core InMemory not a substitute for a relational database?**  
Its query and transaction behavior differs from relational providers, so it cannot prove that SQL behavior is correct.

**72. What should an API integration test assert?**  
Check the status, response contract, and important side effects, including meaningful error and authorization behavior.

## 19. Performance, health checks, and deployment

**73. Why measure before optimizing?**  
Measurement identifies the actual bottleneck and provides a way to check whether a change helped.

**74. What is the difference between liveness and readiness?**  
Liveness asks whether the process should keep running. Readiness asks whether it can safely receive traffic, including required dependencies.

**75. Why can in-memory cache behave differently after scaling out?**  
Each application instance has a separate cache. A change made in one process does not automatically appear in another.

**76. What are forwarded headers for?**  
A trusted reverse proxy can pass the original client address and request scheme. The app must trust only known proxies or networks.

**77. What does dotnet publish create?**  
It produces application files for deployment. Framework-dependent output uses an installed runtime; self-contained output includes a runtime for a selected platform.

**78. Why keep a .NET app on current patches?**  
Supported patches include servicing fixes. Microsoft support policy requires staying current within a supported release.

## Quick review

Before building or deploying a feature, ask:

- What request and response contract does it use?
- Where should validation, business rules, and authorization live?
- Which service lifetime matches the work?
- What does the user see when the resource is missing or input is invalid?
- How will this behavior be tested and monitored?
- Which official documentation page applies to the target framework version?
