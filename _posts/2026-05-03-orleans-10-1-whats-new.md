---
layout: post
title: "Orleans 10.1.0: SQLite Persistence, MemoryPack Serializer, and the New IServiceLifecycle"
date: 2026-05-03
categories: dotnet distributed-systems orleans
---

Orleans 10.1.0 dropped on April 17, 2026 — and this release is heavier than it looks. Thirty-five pull requests landed, but three stand out for working developers: **native SQLite ADO.NET persistence**, the **MemoryPack serializer**, and the new **IServiceLifecycle interface**. Here's what they mean in practice.

## 1. SQLite ADO.NET Persistence — Lightweight Grain Storage Without Azure

For years, Orleans grain persistence defaulted to memory (dev only), Azure Table/Blob Storage, AWS DynamoDB, or a handful of other heavy options. If you wanted a simple relational store without cloud lock-in, you were rolling your own provider.

No longer. PR #9846 adds first-class ADO.NET persistence support for SQLite:

```csharp
// In your silo builder
builder.Services.AddOrleans(silo =>
{
    silo.AddAdoNetGrainStorage("Default", options =>
    {
        options.ConnectionString = "Data Source=orleans.db";
        // Uses Microsoft.Data.SqlClient — not the legacy System.Data.SqlClient
    });
});

// A grain persisting its state to SQLite
public class OrderGrain : Grain<OrderState>, IOrderGrain
{
    public async Task UpdateStatusAsync(string status)
    {
        this.State.Status = status;
        await this.WriteStateAsync(); // Persists to SQLite via ADO.NET
    }
}

public class OrderState
{
    public string OrderId { get; set; }
    public string Status { get; set; }
    public DateTime UpdatedAt { get; set; }
}
```

This works with any ADO.NET-compatible database — SQLite is the first, but the provider is generic. Swap the connection string and you're on PostgreSQL, SQL Server, or MySQL with the same API.

## 2. MemoryPack Serializer — Faster Binary Serialization

PR #9838 introduces MemoryPack as a built-in Orleans serializer. MemoryPack is one of the fastest binary serializers available for .NET — compact, zero-reflection, and GC-friendly.

```csharp
// Switch grain serialization to MemoryPack
builder.Services.AddOrleans(silo =>
{
    silo.AddMemoryPackGrainSerializer();
});

// MemoryPack is also now usable as a general-purpose serializer
// via Orleans.Serialization.MemoryPack
using var stream = new MemoryStream();
MemoryPackSerializer.Serialize(ref stream, myGrainState);
stream.Position = 0;
var restored = MemoryPackSerializer.Deserialize<MyState>(ref stream);
```

Why this matters for Orleans: grain state crosses process boundaries on every call. Faster serialization means lower latency and fewer allocations — directly measurable in high-throughput scenarios like gaming, IoT, or real-time collaboration.

## 3. IServiceLifecycle — Lifecycle Hooks for Client and Silo

PR #9792 adds a first-class lifecycle interface for both client and silo, replacing scattered lifecycle patterns with a unified API:

```csharp
// Implement lifecycle hooks cleanly
public class MyService : IServiceLifecycle
{
    public async Task OnInitializeAsync(CancellationToken ct)
    {
        // Called during service initialization — set up connections, caches
    }

    public async Task OnServicesStartedAsync(CancellationToken ct)
    {
        // Called after all services are started — safe to begin operations
    }

    public async Task OnStopAsync(CancellationToken ct)
    {
        // Graceful shutdown — flush buffers, close connections
    }
}

// Register with DI — Orleans handles the lifecycle
builder.Services.AddSingleton<IServiceLifecycle, MyService>();
```

Before this, lifecycle management was scattered across silo builder callbacks, client builder callbacks, and various extension points. IServiceLifecycle gives you one clean interface that works on both sides.

## Bonus: ConfigureAwait Analyzer (ORLEANS0014)

PR #9845 adds an analyzer that flags `ConfigureAwait(false)` misuse in grain code. Grain methods execute in Orleans' own task scheduler — `ConfigureAwait` is unnecessary and can mask scheduling bugs:

```csharp
// ❌ Flagged by ORLEANS0014 — unnecessary ConfigureAwait in grain code
public async Task<string> GetDataAsync()
{
    var result = await _externalService.GetAsync().ConfigureAwait(false);
    return result;
}

// ✅ Correct — no ConfigureAwait in grain methods
public async Task<string> GetDataAsync()
{
    var result = await _externalService.GetAsync();
    return result;
}
```

This is the kind of analyzer that prevents subtle production bugs — especially when grain code accidentally waits on `TaskScheduler.Default` instead of the Orleans scheduler.

## What Didn't Change

This release is a "polish and harden" release rather than an architectural shift. Notable: the breaking change in v10.0 that replaced `System.Data.SqlClient` with `Microsoft.Data.SqlClient` carries forward — your existing code using the old client will break on upgrade.

## Upgrade Notes

If you're moving to 10.1.0:

1. **Update the SDK:** Requires .NET 10.0.103+ (or .NET 8 for older projects)
2. **Check SqlClient:** Replace any `System.Data.SqlClient` references with `Microsoft.Data.SqlClient`
3. **Try SQLite:** Great option for single-silo deployments or local development without cloud dependencies
4. **Register lifecycle services:** If you have scattered init/start/stop code, consolidate into `IServiceLifecycle` implementations

---

*Running Orleans in production? What's your persistence layer — and have you tried moving off memory?*