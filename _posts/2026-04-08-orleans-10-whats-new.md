---
layout: post
title: "Orleans 10.0: What's New in Microsoft's Distributed Actor Framework"
date: 2026-04-08
categories: dotnet releases
---

Microsoft Orleans 10.0 landed in February 2026, bringing significant improvements to the virtual actor framework. Here's what matters for your distributed .NET applications.

## Highlights of Orleans 10.0

### Built-in Dashboard

Real-time monitoring is now first-class:

```csharp
builder.Host.UseOrleans(silo =>
{
    silo.UseLocalhostClustering()
        .AddDashboard();  // New in v10!
});
```

Access it at `http://localhost:8080` to see cluster health, grain counts, and silo metrics.

### Redis Providers (Stable)

Production-ready Redis support for:
- **Clustering** — Service discovery and membership
- **Persistence** — Grain state storage
- **Reminders** — Persistent timers

```csharp
silo.UseRedisClustering(options => 
    options.ConnectionString = "localhost:6379");
```

### Native CancellationToken Support

Grain methods now fully support cooperative cancellation:

```csharp
public async Task ProcessAsync(CancellationToken ct)
{
    for (var i = 0; i < iterations; i++)
    {
        ct.ThrowIfCancellationRequested();
        await DoWorkAsync();
    }
}
```

## Orleans 9.x Backward Compatibility

Orleans 9.2 (July 2025) introduced several features that 10.0 builds upon:

| Feature | Description |
|---------|-------------|
| **Resource-Optimized Placement** | CPU/memory-aware load balancing |
| **Activation Rebalancing** | Automatic grain redistribution |
| **Log-Structured Storage** | Fewer writes for append-heavy workloads |
| **Strong-Consistency Directory** | Distributed grain location tracking |

## Migration from Orleans 7/8

The good news: Orleans 10.0 maintains API compatibility with 9.x. Most projects just need a package version bump:

```bash
dotnet add package Microsoft.Orleans.Core
dotnet add package Microsoft.Orleans.Server
```

**Breaking change:** Orleans 10.0 requires .NET 8.0 minimum. .NET 6 support is dropped.

## Performance Improvements

Orleans 10.0 includes:
- Faster membership protocol (failure detection in 90 seconds vs 10 minutes)
- Memory-based activation shedding under pressure
- Improved grain placement strategies

## Get Started

```bash
dotnet new orleans silo -n MyOrleansApp
cd MyOrleansApp
dotnet run
```

The template sets up a complete project with clustering, storage, and health checks.

---

Orleans has come a long way since its Microsoft Research origins. With .NET Aspire integration and cloud-native deployment support, it's a solid choice for modern distributed systems.
