---
layout: post
title: "Getting Started with Microsoft Orleans: Virtual Actors for Distributed .NET"
date: 2026-04-08
categories: dotnet distributed-systems
---

Building distributed systems is hard. You have to think about scaling, failure handling, message routing, and state management across multiple servers. Microsoft Orleans makes this feel like regular object-oriented programming.

## What is Orleans?

Orleans is a cross-platform framework that brings the **Virtual Actor model** to .NET. Think of it as "Distributed .NET" — you write grains (objects), host them in silos (processes), and the runtime handles the rest:

- **Location transparency** — Grains are always addressable, regardless of where they run
- **Automatic activation** — Grains wake up on first call, sleep when idle
- **State persistence** — Built-in support for storage providers
- **Load balancing** — Smart placement based on CPU/memory metrics

## A Simple Grain Example

```csharp
// Grain interface (shared between client and server)
public interface ICounterGrain : IGrainWithStringKey
{
    Task IncrementAsync();
    Task<int> GetValueAsync();
}

// Grain implementation
public class CounterGrain : Grain, ICounterGrain
{
    private int _value;

    public Task IncrementAsync() => Task.FromResult(_value++);

    public Task<int> GetValueAsync() => Task.FromResult(_value);
}
```

That's it. No boilerplate, no explicit threading, no distributed transaction handling.

## Setting Up a Cluster

With .NET 9 minimal hosting, Orleans integrates seamlessly:

```csharp
var builder = WebApplication.CreateSlimBuilder(args);

builder.Host.UseOrleans(silo =>
{
    silo.UseLocalhostClustering()
        .AddMemoryGrainStorage("Default")
        .ConfigureLogging(logging => logging.AddConsole());
});

var app = builder.Build();
app.MapGet("/", () => "Orleans is running!");
await app.RunAsync();
```

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Grain** | Virtual actor — the fundamental building block |
| **Silo** | Server process that hosts grains |
| **Cluster** | Group of silos working together |
| **Grain Key** | Unique identity (Guid, int, or string) |

## When to Use Orleans

- ✅ High-scale services (gaming, IoT, real-time apps)
- ✅ Stateful microservices
- ✅ Systems requiring automatic failover
- ❌ Simple CRUD applications
- ❌ Low-latency requirements under 1ms

## Next Steps

Orleans 10.0 (February 2026) supports .NET 8, 9, and 10. The framework powers Microsoft 365, Xbox, and Azure IoT at scale.

Ready to dive deeper? Check out the [official documentation](https://learn.microsoft.com/en-us/dotnet/orleans/overview) and Microsoft's sample projects.
