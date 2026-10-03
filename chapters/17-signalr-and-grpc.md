[Back to notes index](../README.md)

| [Previous: Static files, uploads, and downloads](16-static-files-uploads-and-downloads.md) | [Notes index](../README.md) | [Next: Testing ASP.NET Core apps](18-testing-aspnet-core-apps.md) |
| --- | --- | --- |

# 17. SignalR and gRPC

## SignalR for live updates

SignalR lets a server send messages to connected clients. A hub is a high-level endpoint for client and server calls. It can use WebSockets when available and other transports when needed.

Register and map a hub:

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

A hub method can add a connection to a group. A server-side service can later send a message to that group through IHubContext. Do not store long-lived application state in a hub instance. Hubs are created for calls and connections may be served by different app instances.

Every hub method callable by a client is an input boundary. Authenticate callers, verify they may join a group, validate identifiers, and avoid allowing clients to broadcast trusted state changes directly.

## gRPC for typed service calls

gRPC uses Protocol Buffers contracts and generated client and server code. It is a good fit for efficient service-to-service communication when both ends can use the gRPC protocol. Browser clients may need gRPC-Web or a separate browser-facing API.

A contract describes the service and message shape:

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

ASP.NET Core generates service base types from the contract. The app registers gRPC services and maps an implementation with MapGrpcService. Keep the protocol contract versioned carefully because clients and servers may be deployed at different times.

gRPC supports unary calls, client streaming, server streaming, and bidirectional streaming. Use the simplest shape that fits the communication.

## Choose the communication style

- Use ordinary HTTP APIs for broad client compatibility and resource-oriented operations.
- Use SignalR when connected clients need low-latency updates or two-way messages.
- Use gRPC for strongly typed calls between services where its transport and tooling fit.

These choices can coexist. A browser-facing API may update data and a SignalR event may notify connected clients that the list changed.

## Practice check

Create a hub with one read-only notification event and connect a local browser client. Add server-side authorization before allowing a connection to a private group. For gRPC, define one request and response in a proto file and inspect the generated service base class.

## Common mistakes

- Treating a group name as proof that a user may see the group's messages.
- Keeping authoritative data only in a hub instance.
- Letting a client send trusted state changes without server validation.
- Choosing gRPC for a browser client without checking browser transport support.
- Changing a protobuf field number after clients depend on it.

## References

- [SignalR in ASP.NET Core](https://learn.microsoft.com/aspnet/core/signalr/introduction?view=aspnetcore-10.0)
- [SignalR hubs](https://learn.microsoft.com/aspnet/core/signalr/hubs?view=aspnetcore-10.0)
- [gRPC services in ASP.NET Core](https://learn.microsoft.com/aspnet/core/grpc/?view=aspnetcore-10.0)
