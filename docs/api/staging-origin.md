---
layout: default
title: Staging Origin API
parent: API Reference
nav_order: 5
permalink: /docs/api/staging-origin/
---

# `LocalStagingOrigin` Class
{: .no_toc }

Namespace: `Decnet.PatchProof.Origins`  
Assembly: `Decnet.PatchProof.Origins.dll`
{: .lead }

The `LocalStagingOrigin` class embeds a high-performance HTTP web server inside Unity for serving Addressables content over loopback.

---

## Class Definition

```csharp
public class LocalStagingOrigin : IDisposable
{
    public string BindIp { get; }
    public int Port { get; }
    public string BaseUrl { get; }
    public bool IsRunning { get; }
    public FaultProfile ActiveFaultProfile { get; set; }

    public LocalStagingOrigin(string bindIp = "127.0.0.1", int port = 8899);

    public void MapRoute(string urlPrefix, string localDirectory);
    public void ClearRoutes();
    public bool Start(out string error);
    public void Stop();
    public List<HttpTrace> GetTraces();
    public void ClearTraces();
}
```

---

## Example: Programmatic Staging Server

```csharp
using Decnet.PatchProof.Origins;
using Decnet.PatchProof.Core;

// Initialize server on localhost port 8899
using var origin = new LocalStagingOrigin("127.0.0.1", 8899);

// Mount content directories to virtual routes
origin.MapRoute("/content/v1.0.0/", "D:/Builds/v1.0.0/ServerData");
origin.MapRoute("/content/v2.0.0/", "D:/Builds/v2.0.0/ServerData");

// Attach an active fault profile
origin.ActiveFaultProfile = new FaultProfile
{
    name = "LatencyProfile",
    enabled = true,
    rules = {
        new FaultRule {
            faultType = FaultType.LatencyDelay,
            delayMilliseconds = 1000,
            urlPattern = ".*"
        }
    }
};

// Start server
if (origin.Start(out string startError))
{
    Debug.Log($"Staging origin listening at {origin.BaseUrl}");
}
```
