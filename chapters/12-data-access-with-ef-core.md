[Back to notes index](../README.md)

| [Previous: Blazor Web Apps](11-blazor-web-apps.md) | [Notes index](../README.md) | [Next: Authentication, Identity, and cookies](13-authentication-identity-and-cookies.md) |
| --- | --- | --- |

# 12. Data access with Entity Framework Core

## DbContext and entities

Entity Framework Core maps .NET objects to a database. A DbContext represents a unit of work and tracks changes until they are saved. In a web app, register it with AddDbContext so each request receives a scoped instance.

A small context can expose a set of reading items:

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

Install a provider that matches the database, such as Microsoft.EntityFrameworkCore.Sqlite for SQLite. Register the context:

```csharp
var connectionString =
    builder.Configuration.GetConnectionString("ReadingList");

builder.Services.AddDbContext<ReadingDbContext>(options =>
    options.UseSqlite(connectionString));
```

Add the namespace for the selected provider. Keep credentials and production connection strings in protected configuration.

The response type used in the projections can be a small record:

```csharp
public sealed record ReadingItemResponse(int Id, string Title, bool IsRead);
```

## Query asynchronously

A read-only query does not need change tracking:

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

Projection returns only the fields the API needs. It also prevents a database entity from accidentally becoming the public response contract. Pass CancellationToken to database calls so work can stop when the request is cancelled.

For larger result sets, use paging and apply ordering before Skip and Take. Avoid loading an unbounded table into memory.

## Save changes

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

SaveChangesAsync writes tracked changes. For relational providers, EF Core uses a transaction for the save operation when one is needed. Use an explicit transaction when a larger operation must be atomic across multiple saves or data stores.

## Migrations

A migration records a schema change in source control:

```bash
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```

Install the design package and provider at a compatible major version. Review generated migrations before applying them. Local database updates are convenient in development. Production schema changes need a controlled deployment plan, backup strategy, and an account with only the permissions required.

## Common mistakes

- Sharing one DbContext instance across requests.
- Running synchronous database calls from asynchronous request handlers.
- Returning tracked entities directly from public endpoints.
- Calling ToList before applying filters, sorting, and paging.
- Applying unreviewed migrations directly to a production database.
- Building SQL by concatenating untrusted values.

## Practice check

Create a local SQLite database, add a migration, save two reading items, and query them with AsNoTracking. Change the entity schema, inspect the generated migration, and apply it to a disposable local database.

## References

- [Entity Framework Core documentation](https://learn.microsoft.com/ef/core/)
- [DbContext configuration in ASP.NET Core](https://learn.microsoft.com/ef/core/dbcontext-configuration/)
- [Migrations overview](https://learn.microsoft.com/ef/core/managing-schemas/migrations/)

