---
layout: default
title: Byte-Boundary Faults
parent: Core Concepts
nav_order: 5
permalink: /docs/concepts/byte-boundary-faults/
---

# Byte-Boundary Fault Injection
{: .no_toc }

How PatchProof intercepts HTTP streams to inject real-world network anomalies at precise byte offsets.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Beyond Mocking: Real Wire Simulation

Most testing frameworks simulate network failure by configuring a mock class to return an immediate error response. While useful for UI testing, this misses the hardest bugs in asset streaming:
* What happens when the connection drops **after** the 32 KB catalog header has already been parsed?
* How does the Unity Web Request engine behave when a 50 MB bundle streams 45 MB and then the TCP socket abruptly sends an `RST` packet?
* Does your custom progress bar crash when byte counters stall mid-stream?

PatchProof's embedded [`LocalStagingOrigin`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Origins/LocalStagingOrigin.cs) operates as a genuine HTTP server. When a client requests a file, the server streams the physical bytes from disk through a programmable pipeline that triggers faults at specified thresholds.

---

## Supported Fault Primitives

```
+-----------------------------------------------------------------------------+
|                               Fault Engine                                  |
+-----------------------------------------------------------------------------+
|   1. Timeout / Latency    | Injects delay before headers or payload.        |
|   2. Connection Abort     | Terminates TCP socket at byte offset N.         |
|   3. HTTP Status Codes    | Injects 404 Not Found, 403, or 500.             |
|   4. Payload Corruption   | Flips random bits across a byte range.          |
|   5. Stream Truncation    | Closes connection prematurely with 200 OK header|
+-----------------------------------------------------------------------------+
```

### 1. Bounded Timeout & Latency (`FaultType.Timeout` / `LatencyDelay`)
* **Behavior:** The staging origin deliberately stalls the HTTP response for a specified duration (e.g., 3,000 ms).
* **Use Case:** Verifying that your game's timeout configurations fire correctly and that user interfaces display actionable "Reconnecting..." messages rather than freezing the main thread.

### 2. Connection Drop (`FaultType.ConnectionDrop`)
* **Behavior:** The server transmits valid data up to `triggerAtByteOffset`, then immediately terminates the client socket:
  ```csharp
  // From LocalStagingOrigin.cs
  if (rule.faultType == FaultType.ConnectionDrop && totalBytesSent >= rule.triggerAtByteOffset)
  {
      context.Response.Abort(); // Abrupt TCP RST
      return;
  }
  ```
* **Use Case:** Testing subway tunnel disconnections, elevator WiFi drops, and mobile carrier handoffs.

### 3. HTTP Errors (`FaultType.NotFound404` / `ServerError500`)
* **Behavior:** Intercepts requests matching specific URL regex patterns (e.g., `.*_scenes_.*\.bundle`) and returns configured HTTP status codes with empty or diagnostic error bodies.
* **Use Case:** Testing incomplete CDN rollouts where new catalog metadata points to bundles that have not yet replicated across all edge nodes.

### 4. Payload Corruption (`FaultType.CorruptPayload`)
* **Behavior:** The server streams the requested file, but when it reaches `triggerAtByteOffset`, it inverts bits across `corruptByteCount` bytes before resuming transmission.
* **Use Case:** Confirms that Unity's CRC / SHA checksum verification detects corrupted archives and rejects them before passing bad data into memory.

### 5. Stream Truncation (`FaultType.Truncation`)
* **Behavior:** The HTTP response advertises a full `Content-Length` (e.g., 10,000,000 bytes), but the server closes the stream after sending only 2,000,000 bytes.
* **Use Case:** Exposes unhandled stream EOF exceptions and validates archive download resumability.

---

## Precise URL Pattern Matching

Fault rules use URL regex pattern matching to target specific asset categories:

| Target Pattern | Intended Scope | Example Real-World Scenario |
|:---|:---|:---|
| `\.hash$` | Catalog Hash Files | Simulates CDN returning stale hash when checking for updates |
| `catalog_.*\.json$` | Catalog Manifests | Simulates broken or truncated catalog downloads |
| `.*_characters_.*\.bundle$` | Character Asset Bundles | Isolates failures to optional gameplay bundles |
| `.*` | All HTTP Requests | Simulates complete server downtime |

> ### Pro Tip: Request Trace Telemetry
> Every triggered fault is recorded in the scenario's request trace timeline, documenting the exact byte offset where the fault occurred and the client's subsequent retry or failure behavior.
