[Back to notes index](../README.md)

| [Previous: Performance, health checks, and deployment](19-performance-health-and-deployment.md) | [Notes index](../README.md) | [Next: Complete questions and answers](99-complete-q-and-a.md) |
| --- | --- | --- |

# 98. All code samples

This chapter gathers every fenced code sample from the core chapters in one place. Each section links back to the notes that explain the sample.

## 1. ASP.NET Core and HTTP basics

[Open chapter](01-aspnet-core-and-http.md)

### Sample 1

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => Results.Text("Reading list service"));
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));

app.Run();
```

### Sample 2

```bash
curl -i https://localhost:5001/health
```

## 2. .NET setup and project structure

[Open chapter](02-dotnet-setup-and-project-structure.md)

### Sample 1

```bash
dotnet --info
dotnet --list-sdks
dotnet --list-runtimes
```

### Sample 2

```bash
dotnet new webapi -n ReadingList.Api
cd ReadingList.Api
dotnet run
```

### Sample 3

```bash
dotnet new webapi --use-controllers -n ReadingList.Api
```

### Sample 4

```bash
dotnet new webapp -n ReadingList.Web
dotnet new blazor -n ReadingList.Blazor
```

### Sample 5

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

### Sample 6

```bash
dotnet restore
dotnet build
dotnet run
dotnet watch
dotnet test
```

### Sample 7

```bash
dotnet new sln -n ReadingList
dotnet sln ReadingList.sln add src/ReadingList.Api/ReadingList.Api.csproj
dotnet sln ReadingList.sln add tests/ReadingList.Api.Tests/ReadingList.Api.Tests.csproj
```

## 3. Hosting, environments, and configuration

[Open chapter](03-hosting-environments-and-configuration.md)

### Sample 1

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services and configure the app here.

var app = builder.Build();

// Configure middleware and map endpoints here.

app.Run();
```

### Sample 2

```json
{
  "ReadingList": {
    "PageSize": 20,
    "AllowGuestReads": true
  }
}
```

### Sample 3

```csharp
var pageSize = builder.Configuration.GetValue<int>("ReadingList:PageSize", 20);
```

### Sample 4

```csharp
if (builder.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

### Sample 5

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:ReadingList" "replace-with-a-local-value"
dotnet user-secrets list
```

## 4. Middleware and the request pipeline

[Open chapter](04-middleware-and-request-pipeline.md)

### Sample 1

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

### Sample 2

```csharp
app.UseMiddleware<RequestLogMiddleware>();
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));
```

### Sample 3

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

## 5. Routing and endpoint design

[Open chapter](05-routing-and-endpoints.md)

### Sample 1

```csharp
app.MapGet("/reading-items/{id:int}", (int id) =>
    Results.Ok(new { Id = id, Title = "Distributed systems notes" }));
```

### Sample 2

```csharp
var readingItems = app.MapGroup("/api/reading-items")
    .WithTags("Reading items");

readingItems.MapGet("/", () =>
    Results.Ok(new[] { "Distributed systems notes", "HTTP notes" }));

readingItems.MapGet("/{id:int}", (int id) =>
    Results.Ok(new { Id = id, Title = "Distributed systems notes" }));
```

### Sample 3

```csharp
app.MapGet("/api/reading-items/{id:int}", (int id) =>
        Results.Ok(new { Id = id }))
    .WithName("GetReadingItem");
```

## 6. Dependency injection, options, and logging

[Open chapter](06-dependency-injection-options-and-logging.md)

### Sample 1

```csharp
builder.Services.AddScoped<IReadingItemService, ReadingItemService>();
```

### Sample 2

```csharp
public sealed class ReadingItemService(IReadingItemStore store)
{
    public Task<IReadOnlyList<ReadingItem>> GetAllAsync(
        CancellationToken cancellationToken) =>
        store.GetAllAsync(cancellationToken);
}
```

### Sample 3

```csharp
public sealed class ReadingListOptions
{
    public const string SectionName = "ReadingList";

    public int PageSize { get; set; } = 20;
}
```

### Sample 4

```csharp
builder.Services
    .AddOptions<ReadingListOptions>()
    .Bind(builder.Configuration.GetSection(ReadingListOptions.SectionName))
    .Validate(options => options.PageSize is > 0 and <= 100,
        "PageSize must be between 1 and 100.")
    .ValidateOnStart();
```

### Sample 5

```csharp
app.MapGet("/api/reading-items", (
    ILogger<Program> logger,
    IReadingItemService service,
    CancellationToken cancellationToken) =>
{
    logger.LogInformation("Reading the current item collection");
    return service.GetAllAsync(cancellationToken);
});
```

## 7. Minimal APIs

[Open chapter](07-minimal-apis.md)

### Sample 1

```csharp
using Microsoft.AspNetCore.Http.HttpResults;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenApi();

var app = builder.Build();
var items = new List<ReadingItem>
{
    new(1, "HTTP fundamentals", false),
    new(2, "Dependency injection", true)
};

var routes = app.MapGroup("/api/reading-items")
    .WithTags("Reading items");

routes.MapGet("/", () => TypedResults.Ok(items));

routes.MapGet("/{id:int}", (int id) => FindItem(id, items));

routes.MapPost("/", (CreateReadingItem request) =>
{
    var item = new ReadingItem(
        items.Count == 0 ? 1 : items.Max(existing => existing.Id) + 1,
        request.Title,
        false);

    items.Add(item);
    return TypedResults.Created($"/api/reading-items/{item.Id}", item);
});

app.MapOpenApi();
app.Run();

static Results<Ok<ReadingItem>, NotFound> FindItem(
    int id,
    List<ReadingItem> items)
{
    var item = items.SingleOrDefault(existing => existing.Id == id);
    if (item is null)
    {
        return TypedResults.NotFound();
    }

    return TypedResults.Ok(item);
}

public sealed record ReadingItem(int Id, string Title, bool IsRead);
public sealed record CreateReadingItem(string Title);
```

## 8. Controllers, REST, and OpenAPI

[Open chapter](08-controllers-rest-and-openapi.md)

### Sample 1

```csharp
builder.Services.AddControllers();
builder.Services.AddOpenApi();

var app = builder.Build();

app.MapOpenApi();
app.MapControllers();
```

### Sample 2

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

### Sample 3

```csharp
public sealed record CreateReadingItemRequest(
    [Required, StringLength(120)] string Title);
```

## 9. Model binding, validation, and filters

[Open chapter](09-model-binding-validation-and-filters.md)

### Sample 1

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

### Sample 2

```csharp
public sealed class CreateReadingItemRequest
{
    [Required]
    [StringLength(120, MinimumLength = 2)]
    public string Title { get; set; } = string.Empty;
}
```

## 10. Razor Pages and MVC

[Open chapter](10-razor-pages-and-mvc.md)

### Sample 1

```html
@page
@model CreateModel

<form method="post">
    <label asp-for="Input.Title"></label>
    <input asp-for="Input.Title" />
    <span asp-validation-for="Input.Title"></span>
    <button type="submit">Add item</button>
</form>
```

### Sample 2

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

### Sample 3

```csharp
public sealed class CreateModel(IReadingItemService service) : PageModel
{
    [BindProperty]
    public CreateReadingItemRequest Input { get; set; } = new();

    public async Task<IActionResult> OnPostAsync(
        CancellationToken cancellationToken)
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }

        await service.CreateAsync(Input.Title, cancellationToken);
        return RedirectToPage("Index");
    }
}
```

### Sample 4

```csharp
public sealed class ReadingItemsController(
    IReadingItemService service) : Controller
{
    public async Task<IActionResult> Index(
        CancellationToken cancellationToken)
    {
        var items = await service.GetAllAsync(cancellationToken);
        return View(items);
    }
}
```

## 11. Blazor Web Apps

[Open chapter](11-blazor-web-apps.md)

### Sample 1

```csharp
builder.Services
    .AddRazorComponents()
    .AddInteractiveServerComponents();

var app = builder.Build();

app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();
```

### Sample 2

```razor
@page "/counter"
@rendermode InteractiveServer

<PageTitle>Counter</PageTitle>

<h1>Reading list counter</h1>
<p>Items added: @count</p>
<button @onclick="AddOne">Add one</button>

@code {
    private int count;

    private void AddOne()
    {
        count++;
    }
}
```

## 12. Data access with Entity Framework Core

[Open chapter](12-data-access-with-ef-core.md)

### Sample 1

```csharp
public sealed class ReadingDbContext(
    DbContextOptions<ReadingDbContext> options) : DbContext(options)
{
    public DbSet<ReadingItem> ReadingItems => Set<ReadingItem>();
}

public sealed class ReadingItem
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public bool IsRead { get; set; }
}
```

### Sample 2

```csharp
var connectionString =
    builder.Configuration.GetConnectionString("ReadingList");

builder.Services.AddDbContext<ReadingDbContext>(options =>
    options.UseSqlite(connectionString));
```

### Sample 3

```csharp
app.MapGet("/api/reading-items", async (
    ReadingDbContext db,
    CancellationToken cancellationToken) =>
{
    var items = await db.ReadingItems
        .AsNoTracking()
        .OrderBy(item => item.Title)
        .Select(item => new ReadingItemResponse(
            item.Id,
            item.Title,
            item.IsRead))
        .ToListAsync(cancellationToken);

    return TypedResults.Ok(items);
});
```

### Sample 4

```csharp
app.MapPost("/api/reading-items", async (
    CreateReadingItemRequest request,
    ReadingDbContext db,
    CancellationToken cancellationToken) =>
{
    var item = new ReadingItem { Title = request.Title };
    db.ReadingItems.Add(item);
    await db.SaveChangesAsync(cancellationToken);

    var response = new ReadingItemResponse(
        item.Id,
        item.Title,
        item.IsRead);

    return TypedResults.Created(
        $"/api/reading-items/{item.Id}",
        response);
});
```

### Sample 5

```bash
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## 13. Authentication, Identity, and cookies

[Open chapter](13-authentication-identity-and-cookies.md)

### Sample 1

```csharp
builder.Services
    .AddDefaultIdentity<IdentityUser>(options =>
    {
        options.SignIn.RequireConfirmedAccount = true;
        options.Password.RequiredLength = 12;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### Sample 2

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

## 14. Authorization and web security

[Open chapter](14-authorization-and-web-security.md)

### Sample 1

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("ReadingItemEditor", policy =>
        policy.RequireRole("Editor"));
```

### Sample 2

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

## 15. Error handling, Problem Details, and diagnostics

[Open chapter](15-errors-problem-details-and-diagnostics.md)

### Sample 1

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

### Sample 2

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

## 16. Static files, uploads, and downloads

[Open chapter](16-static-files-uploads-and-downloads.md)

### Sample 1

```csharp
app.UseStaticFiles();
```

### Sample 2

```csharp
[HttpPost("uploads")]
[RequestSizeLimit(5_000_000)]
public async Task<IActionResult> Upload(
    IFormFile file,
    CancellationToken cancellationToken)
{
    if (file.Length == 0)
    {
        return BadRequest("Choose a non-empty file.");
    }

    var extension = Path.GetExtension(file.FileName);
    var allowedExtensions = new HashSet<string>(
        StringComparer.OrdinalIgnoreCase)
    {
        ".pdf",
        ".txt"
    };

    if (!allowedExtensions.Contains(extension))
    {
        return BadRequest("File type is not allowed.");
    }

    var storedName = $"{Guid.NewGuid():N}{extension}";
    var path = Path.Combine(_uploadDirectory, storedName);

    await using var stream = System.IO.File.Create(path);
    await file.CopyToAsync(stream, cancellationToken);

    return Ok(new { storedName });
}
```

### Sample 3

```csharp
return PhysicalFile(
    storedPath,
    "application/pdf",
    downloadName: "reading-notes.pdf",
    enableRangeProcessing: true);
```

## 17. SignalR and gRPC

[Open chapter](17-signalr-and-grpc.md)

### Sample 1

```csharp
builder.Services.AddSignalR();

var app = builder.Build();

app.MapHub<ReadingListHub>("/hubs/reading-list");

public sealed class ReadingListHub : Hub
{
    public Task JoinList(string listId)
    {
        return Groups.AddToGroupAsync(
            Context.ConnectionId,
            $"list-{listId}");
    }
}
```

### Sample 2

```proto
syntax = "proto3";

package readinglist;

service ReadingList {
  rpc GetItem (GetItemRequest) returns (ReadingItemReply);
}

message GetItemRequest {
  int32 id = 1;
}

message ReadingItemReply {
  int32 id = 1;
  string title = 2;
  bool is_read = 3;
}
```

## 18. Testing ASP.NET Core apps

[Open chapter](18-testing-aspnet-core-apps.md)

### Sample 1

```csharp
public partial class Program;
```

### Sample 2

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

### Sample 3

```bash
dotnet test
dotnet test --filter FullyQualifiedName~HealthEndpointTests
```

## 19. Performance, health checks, and deployment

[Open chapter](19-performance-health-and-deployment.md)

### Sample 1

```csharp
builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/health/live");
```

### Sample 2

```bash
dotnet restore
dotnet test
dotnet publish -c Release -o ./publish
```
