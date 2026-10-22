---
layout: default
title: Staging Origin Server
parent: Features Overview
nav_order: 3
permalink: /docs/features/staging-origin/
---

# Staging Origin Server
{: .no_toc }

Embedded, zero-CDN HTTP staging origin with per-run virtual route mapping and credential redaction.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why an Embedded Origin?

Testing content updates against real cloud storage buckets (e.g., AWS S3, Cloudflare R2, Google Cloud Storage, or Unity Cloud Content Delivery) introduces serious risks during local development:
* **Accidental Overwrites:** Pushing an experimental catalog can break internal playtests or active live players.
* **Network Flakiness:** Internet speed variations make latency benchmarks unreproducible.
* **Cost:** Continuous multi-gigabyte re-downloads inflate cloud egress bills.
* **Credential Security:** Exposing API tokens in local scripts or CI environments risks security leaks.

PatchProof's **Staging Origin** ([`LocalStagingOrigin.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Origins/LocalStagingOrigin.cs)) provides a lightweight, dedicated HTTP server that runs entirely inside your Unity Editor process.

---

## Architectural Highlights

```
+-----------------------------------------------------------------------------+
|                         PatchProof Local Origin                             |
|                        http://127.0.0.1:8899/                               |
+-----------------------------------------------------------------------------+
  |-- Route: /content/v1.0.0/   --> Mounts D:/Builds/v1.0.0/ServerData/
  |-- Route: /content/v2.0.0/   --> Mounts D:/Builds/v2.0.0/ServerData/
  |-- Fault Engine Interceptor  --> Evaluates Active FaultProfile rules
  |-- Telemetry Logger          --> Records HttpTrace events in memory
+-----------------------------------------------------------------------------+
```

### 1. Loopback-Only Binding (`127.0.0.1`)
By default, the server binds strictly to the loopback IP (`127.0.0.1`). It refuses connections from outside the host machine, ensuring your unreleased game assets are never exposed over your local office LAN or public WiFi.

### 2. Multi-Route Virtual Mounts
The server can mount multiple physical directories simultaneously:
```csharp
// Dynamically maps URLs to physical folders on disk
_origin.MapRoute("/content/v1.0.0/", baselineContentDirectory);
_origin.MapRoute("/content/v2.0.0/", candidateContentDirectory);
```
This allows the client binary to fetch baseline assets and candidate update assets concurrently during rollback rehearsals without copying files.

### 3. Credential Redaction Pipeline
When interfacing with Unity Cloud Content Delivery (CCD) or remote staging profiles, URLs often carry sensitive query parameters:
```
http://origin.example.com/bundle?token=secret_auth_99182&sig=abc78291
```

PatchProof's [`CcdEndpointAdapter`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Adapters/CcdEndpointAdapter.cs) intercepts all incoming request strings and sanitizes sensitive credentials before logging:
```csharp
// From CcdEndpointAdapter.cs RedactCredentials regex
string pattern = @"([?&](?:token|access_token|key|api_key|auth|signature|secret|sig)=)([^&]+)";
return Regex.Replace(url, pattern, "$1[REDACTED]");
```
The sanitized output appears safely in trace logs:
```
http://origin.example.com/bundle?token=[REDACTED]&sig=[REDACTED]
```

---

## Configuring the Origin in the Editor

1. Open `Tools > PatchProof > Origin & Fault Profiles`.
2. Inspect the **Server Configuration** box:
   * **Bind IP:** Defaults to `127.0.0.1`.
   * **Port:** Defaults to `8899` (Configurable from 1025 to 65534).
   * **Read Buffer Size:** `65,536 bytes` (64 KB).
3. Click **"Test Server Port"** to verify that the port is open and unbound by other applications.
4. If the port is in use, PatchProof automatically increments to the next available ephemeral port.

> ### Pro Tip: High-Concurrency Streaming
> The Local Staging Origin uses asynchronous `BeginGetContext` worker threads. It easily serves multiple simultaneous client instances or multi-threaded Addressables downloaders without choking the Unity main thread.
