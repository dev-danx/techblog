---
layout: post
title: ".NET April 2026 Servicing Updates: What You Need to Patch Now"
date: 2026-04-27
categories: [dotnet, security]
---

The .NET team just released the April 2026 servicing updates, and this one matters — there are security fixes that you'll want to apply before your next deployment.

## What's Updated

The April 2026 release covers:
- **.NET 8.0** (LTS) — security + non-security fixes
- **.NET 9.0** — security + non-security fixes  
- **.NET Framework 4.8** — Windows-only fixes

## Why Servicing Updates Matter

.NET follows a predictable servicing model:
- **Security fixes** → Released immediately, patch ASAP
- **Non-security fixes** → Bundled in monthly updates

The April release includes CVEs that affect production workloads. If you're running .NET 8 or 9 in production, this update should be in your deployment pipeline this week.

## How to Update

### Docker
```dockerfile
# .NET 8
FROM mcr.microsoft.com/dotnet/aspnet:8.0-apache

# .NET 9  
FROM mcr.microsoft.com/dotnet/aspnet:9.0-apache
```

### CLI
```bash
# Check your current version
dotnet --version

# Update SDK (if needed)
dotnet update

# For container deployments, just update your base image tag
```

### Azure DevOps / GitHub Actions
```yaml
- task: UseDotNet@2
  displayName: 'Install .NET 8.0.x'
  inputs:
    packageType: 'sdk'
    version: '8.0.x'
```

## What's Actually Fixed

The security CVEs typically cover:
- **Remote code execution** vulnerabilities
- **Denial of service** vectors
- **Information disclosure** issues

Check the [official release notes](https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-april-2026-servicing-updates/) for the complete CVE list.

## Recommended Action

```bash
# Quick check script for your servers
dotnet --list-sdks
# Verify you're on 8.0.8+ or 9.0.4+ (or whatever the latest is)
```

For containerized apps: rebuild and redeploy with the updated base image.

For VMs: run `apt update && apt upgrade dotnet*` (Linux) or check Windows Update.

---

*Tip: Subscribe to the [.NET Blog](https://devblogs.microsoft.com/dotnet/) to get these updates directly — it's the fastest way to know when a patch affects your stack.*
