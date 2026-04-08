---
layout: post
title: "Building Scalable Orleans Clusters: From Localhost to Kubernetes"
date: 2026-04-08
categories: dotnet devops
---

One of Orleans' killer features is how seamlessly it scales from a single development machine to a multi-node Kubernetes cluster. Let's walk through the journey.

## 1. Local Development

Start with localhost clustering — zero config:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseOrleans(silo =>
{
    silo.UseLocalhostClustering();
});

await builder.Build().RunAsync();
```

Perfect for local development. Grains activate on demand, and there's no infrastructure to manage.

## 2. Multi-Node Local Cluster

Test clustering locally with multiple silos:

```csharp
// Silo 1
silo.UseLocalhostClustering(siloPort: 11111, gatewayPort: 30000);

// Silo 2  
silo.UseLocalhostClustering(siloPort: 11112, gatewayPort: 30001);
```

Connect clients to any gateway:

```csharp
var client = new ClientBuilder()
    .UseLocalhostClustering(gatewayPort: 30000)
    .Build();
```

## 3. Production: Service Discovery

For production, you need dynamic service discovery. Orleans supports multiple backends:

### Azure Table Storage
```csharp
silo.UseAzureTableClustering(options =>
    options.ConnectionString = "your-storage-connection-string");
```

### Redis
```csharp
silo.UseRedisClustering(options =>
    options.ConnectionString = "redis-connection-string");
```

### SQL Server / PostgreSQL
```csharp
silo.UseAdoNetClustering(options =>
{
    options.ConnectionString = "server=db;database=orleans;";
    options.Invariant = "Orleans.Clustering.AdoNet.MySql";
});
```

## 4. Kubernetes Deployment

Deploy to Kubernetes using the Orleans operator or manual configuration:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orleans-silo
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: silo
        image: your-orleans-app:latest
        ports:
        - containerPort: 11111  # Silo port
        - containerPort: 30000  # Gateway port
```

For Kubernetes, use the membership operator:

```csharp
silo.UseKubernetesClustering();
```

## 5. Smart Load Balancing

Orleans 9+ uses **Resource-Optimized Placement** by default:

```csharp
silo.Configure<ResourceOptimizedPlacementOptions>(o =>
{
    o.CpuUsageWeight = 60;
    o.MemoryUsageWeight = 20;
    o.AvailableMemoryWeight = 15;
    o.MaxAvailableMemoryWeight = 5;
});
```

The cluster automatically distributes grains based on real-time resource usage.

## 6. Grain Migration

Moving grains between silos without downtime (great for rolling upgrades):

```csharp
// Request migration when grain is idle
await grain.MigrateOnIdle();

// Custom migration with state transfer
public class MyGrain : Grain, IGrainMigrationParticipant
{
    public void OnDehydrate(IDehydrationContext ctx)
        => ctx.Set("state", this._state);
    
    public void OnRehydrate(IRehydrationContext ctx)
        this._state = ctx.Get<MyState>("state");
}
```

## 7. Observability

Add OpenTelemetry for full observability:

```csharp
silo.UseOpenTelemetry();
```

This traces grain calls, silos, and provides metrics for Prometheus/Grafana.

## Architecture Overview

```
┌─────────────────────────────────────────┐
│              Orleans Cluster            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │  Silo 1 │  │  Silo 2 │  │  Silo 3 │  │
│  │ (Node A)│  │ (Node B)│  │ (Node C)│  │
│  └─────────┘  └─────────┘  └─────────┘  │
└─────────────────────────────────────────┘
          ▲              ▲              ▲
          │   Service Discovery (Redis/Table/SQL)
          │
┌─────────────────────────────────────────┐
│              Clients                    │
│    Web API    │    gRPC    │    Apps   │
└─────────────────────────────────────────┘
```

## Summary

| Environment | Clustering | Use Case |
|-------------|------------|----------|
| Local Dev | Localhost | Testing |
| CI/CD | Localhost (multi-silo) | Integration tests |
| Staging | Single cloud provider | QA |
| Production | Multi-region Kubernetes | Scale to thousands |

The beauty of Orleans? Your application code stays the same — only the hosting configuration changes.
