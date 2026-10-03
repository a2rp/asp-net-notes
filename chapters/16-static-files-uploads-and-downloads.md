[Back to notes index](../README.md)

| [Previous: Error handling, Problem Details, and diagnostics](15-errors-problem-details-and-diagnostics.md) | [Notes index](../README.md) | [Next: SignalR and gRPC](17-signalr-and-grpc.md) |
| --- | --- | --- |

# 16. Static files, uploads, and downloads

## Public static assets

Files under wwwroot are intended to be served to anyone who can reach the application. Typical examples are stylesheets, browser scripts, icons, and public images. Do not place private reports, user documents, configuration files, or secrets in this directory.

For a basic static file setup:

```csharp
app.UseStaticFiles();
```

The middleware serves known file types from the web root. Modern templates can use MapStaticAssets for build-time discovery and optimized delivery of assets. Follow the project template and framework documentation for the chosen approach.

## Accept an uploaded file

Browser forms send files as multipart/form-data. An upload endpoint must impose a size limit and treat every file property as untrusted. A safe first step is to generate a server-side name and store the file outside the public web root:

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

The sample checks a suffix only to demonstrate a rule. A file name and declared content type do not prove what the file contains. Production handling should combine an allow-list, size limits, content inspection, malware scanning where appropriate, safe storage permissions, and authorization.

## Return a download

Do not let a caller pass an arbitrary disk path. Look up a stored file by an application identifier, check access to that record, then return the server-selected file:

```csharp
return PhysicalFile(
    storedPath,
    "application/pdf",
    downloadName: "reading-notes.pdf",
    enableRangeProcessing: true);
```

For small in-memory content, FileContentResult can be appropriate. For large content, stream it and apply request cancellation. Protect private downloads with the same access checks as the record that owns the file.

## Operational details

Upload limits exist at more than one layer. Check application limits, reverse proxy limits, and hosting limits. Use asynchronous stream operations. Avoid loading a large file fully into memory. Consider quotas, retention rules, storage encryption, backup, and deletion when a user removes the associated record.

## Practice check

Serve a public CSS file from wwwroot. Add a separate upload directory outside that folder. Reject empty or oversized uploads, store accepted files with generated names, and require authorization before downloading them.

## Common mistakes

- Saving an untrusted original file name directly to disk.
- Trusting the browser-provided content type as proof.
- Putting private uploads under wwwroot.
- Accepting unlimited file sizes.
- Returning a physical path supplied by the caller.
- Forgetting to remove stored files when their records expire.

## References

- [Static files in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/static-files?view=aspnetcore-10.0)
- [Upload files in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/file-uploads?view=aspnetcore-10.0)
- [Return files from a controller](https://learn.microsoft.com/aspnet/core/mvc/controllers/actions?view=aspnetcore-10.0)
