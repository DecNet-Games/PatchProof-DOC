---
layout: default
title: Fault Injection Engine
parent: Features Overview
nav_order: 4
permalink: /docs/features/fault-injection/
---

# Fault Injection Engine
{: .no_toc }

Configuring and executing deterministic network failures at precise byte boundaries.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Deterministic Faults?

Random network monkey testing can confirm that your game eventually handles errors, but it cannot reproduce the exact condition that caused a crash.

PatchProof's **Fault Injection Engine** allows engineers to write deterministic rules:
* *"Abort the connection on request #2 after exactly 65,536 bytes of `soundtrack.bundle` have streamed."*
* *"Inject a 404 status code when requesting any file matching `.*_scenes_.*\.bundle`."*
* *"Corrupt 128 bytes starting at offset 1,024 to verify CRC rejection."*

These rules execute identically every single time, enabling automated regression prevention.

---

## Fault Profile Structure

Defined in [`FaultProfile.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/CommonTypes.cs):

```csharp
[Serializable]
public class FaultProfile
{
    public string name = "New Fault Profile";
    public bool enabled = true;
    public List<FaultRule> rules = new List<FaultRule>();
}

[Serializable]
public class FaultRule
{
    public FaultType faultType;
    public string urlPattern = ".*";
    public long triggerAtByteOffset = 0;
    public int corruptByteCount = 64;
    public int delayMilliseconds = 0;
    public int httpStatusCode = 500;
}
```

---

## Detailed Fault Types

### 1. Connection Drop (`FaultType.ConnectionDrop`)
* **Mechanism:** Reads and transmits bytes from disk normally. When `totalBytesTransferred >= triggerAtByteOffset`, the socket calls `HttpListenerResponse.Abort()`.
* **Client Experience:** Unity Web Request throws `UnityWebRequestException: Request aborted / Connection reset by peer`.
* **Expected Client Handling:** Client retry loops should resume downloading from byte offset $N$ using HTTP `Range` headers, or restart the bundle download cleanly.

### 2. Timeout & Latency (`FaultType.Timeout` / `LatencyDelay`)
* **Mechanism:** Blocks the origin thread with `Thread.Sleep(delayMilliseconds)` before sending response headers or streaming content.
* **Client Experience:** Unity Web Request hangs until its internal `timeout` expires.
* **Expected Client Handling:** Client must catch `UnityWebRequest.Result.ConnectionError` with timeout description and trigger user-facing reconnect dialogs.

### 3. HTTP 404 Not Found (`FaultType.NotFound404`)
* **Mechanism:** Responds with `StatusCode = 404` and empty content body.
* **Client Experience:** Addressables runtime logs: `RemoteProviderException: Unable to load asset bundle from 404 URL`.
* **Expected Client Handling:** Critical updates must abort cleanly and notify the player of an incomplete rollout.

### 4. Payload Bit-Flipping (`FaultType.CorruptPayload`)
* **Mechanism:** As bytes stream through memory, PatchProof executes a bitwise XOR on `corruptByteCount` bytes:
  ```csharp
  buffer[i] = (byte)(buffer[i] ^ 0xFF);
  ```
* **Client Experience:** The download finishes with 200 OK, but Addressables header parsing throws `CRC Mismatch: The downloaded bundle did not match the expected hash`.
* **Expected Client Handling:** Addressables must purge the corrupted bundle from `UnityEngine.Caching` and not attempt to load broken assets.

---

## Creating Profiles in the Editor

1. Open `Tools > PatchProof > Origin & Fault Profiles`.
2. Click **"Add Fault Profile"**.
3. Choose a template or create custom rules:
   * **Template: Mobile Subway Disconnect:** Drops connection at 256 KB.
   * **Template: Incomplete CDN Edge:** Injects 404 on all `.bundle` files.
   * **Template: High Latency Satellite:** Injects 1,500 ms delay on all requests.
4. Set the **URL Pattern** regex.
5. Click **"Save Fault Profile"**. The profile is immediately available in the Rehearsal Room scenario dropdown.

> ### Pro Tip: Testing Catalog Downgrades
> Create a fault rule targeting `\.hash$` files that returns a 304 Not Modified header or delayed response. This tests whether your game client gracefully handles scenarios where the remote CDN has not yet updated its primary hash file.
