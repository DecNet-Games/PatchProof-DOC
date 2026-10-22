---
layout: default
title: Request Traces & Telemetry
parent: Features Overview
nav_order: 8
permalink: /docs/features/request-traces/
---

# Request Traces & Telemetry
{: .no_toc }

Deep packet inspection, timing watermarks, wire vs disk bytes, and credential sanitation.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Wire Telemetry Matters

Addressables hides HTTP operations behind high-level APIs like `Addressables.DownloadDependenciesAsync()`. While convenient for gameplay programming, this abstraction makes it difficult to diagnose network performance issues:
* Did the client request 4 small bundles concurrently or 1 large bundle sequentially?
* Did an unchanged bundle download over the wire, or did Unity resolve it from local cache?
* What was the exact Time-to-First-Byte (TTFB) for the remote catalog file?

PatchProof's **Request Trace Engine** captures every raw HTTP interaction between the standalone client and the local staging origin.

---

## The `HttpTrace` Data Structure

Defined in [`RunRecords.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/RunRecords.cs), each trace entry records a complete request-response transaction:

```csharp
[Serializable]
public class HttpTrace
{
    public string url;                 // Sanitized URL path
    public string httpMethod;          // GET, HEAD, POST
    public int statusCode;             // 200, 206, 304, 404, 500
    public long wireBytes;             // Actual bytes sent over TCP socket
    public long decompressedBytes;     // Decompressed payload size
    public double durationMs;          // Request latency in milliseconds
    public string clientIp;            // 127.0.0.1
    public string faultInjected;       // None, AbortedAtOffset:131072, etc.
    public long timestampTicks;        // High-resolution timestamp
}
```

---

## The Request Trace Inspector

In the Rehearsal Room window, clicking **"View HTTP Request Traces"** on any scenario tile opens the detailed trace timeline:

```
+---------------------------------------------------------------------------------------------------+
| Request Trace Inspector - Run: run_8f91c2a0 (FreshCache_CleanNetwork)                             |
+---------------------------------------------------------------------------------------------------+
| TIME   | METHOD | STATUS | WIRE BYTES | LATENCY | FAULT | URL PATH                                    |
|--------+--------+--------+------------+---------+-------+---------------------------------------------|
| 0.012s | GET    | 200 OK |      1,024 |    4ms  | None  | /content/v2.0.0/catalog_2026.10.22.hash     |
| 0.028s | GET    | 200 OK |     32,768 |   12ms  | None  | /content/v2.0.0/catalog_2026.10.22.bin      |
| 0.115s | GET    | 200 OK |  4,194,304 |   45ms  | None  | /content/v2.0.0/hero_knight_assets.bundle   |
| 0.180s | GET    | 200 OK |    524,288 |    8ms  | None  | /content/v2.0.0/shared_materials.bundle     |
+---------------------------------------------------------------------------------------------------+
| TOTALS: 4 Requests | Transferred: 4.75 MB | Total Transfer Time: 69ms | Cache Hits: 0             |
+---------------------------------------------------------------------------------------------------+
```

---

## Verifying Cache Hits vs Wire Downloads

When comparing a **Fresh Cache** run against a **Warm Old Cache** run:

### Fresh Cache Trace
Every bundle required by the candidate release appears with `Status: 200 OK` and a positive `Wire Bytes` count matching the physical bundle file size.

### Warm Old Cache Trace
1. The client requests the catalog hash (`200 OK`).
2. The client requests the catalog binary (`200 OK`).
3. For bundles that were unchanged between v1.0.0 and v2.0.0, **zero HTTP requests appear in the trace**.
4. The client's runtime probe records a `CacheHit` event, confirming that Addressables loaded the existing archive directly from `UnityEngine.Caching`.
5. Only genuinely modified bundles are transferred over the wire.

---

## Automatic Redaction Verification

PatchProof enforces strict privacy across all recorded traces. If a remote staging URL carries API keys, access signatures, or private bearer tokens:
* Pattern: `(?i)(?:token|access_token|key|api_key|auth|signature|secret|sig)=[^&]+`
* Replacement: `[REDACTED]`

Trace logs, exported JSON files, and HTML audit summaries never store unredacted credentials.

> ### Pro Tip: Detecting Accidental CDN Hits
> If a standalone player misconfiguration causes Addressables to bypass the local staging origin and attempt to contact a public CDN, the probe detects the external URI and logs an `ExternalNetworkViolation` alert.
