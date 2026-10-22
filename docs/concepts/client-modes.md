---
layout: default
title: Client Execution Modes
parent: Core Concepts
nav_order: 2
permalink: /docs/concepts/client-modes/
---

# Client Execution Modes
{: .no_toc }

Understanding the three execution models in PatchProof: Instrumented Binary, Release Twin, and External Observation.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Multiple Client Modes?

Testing content updates against an older client requires striking a balance between **depth of inspection** and **fidelity to historical code**:
* Can you modify the older project to embed test hooks?
* Do you only have the compiled `.exe` from a release shipped six months ago?
* Are you running in a headless CI container?

PatchProof defines three explicit **Client Modes** in [`HistoricalClientDescriptor.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Adapters/HistoricalClientDescriptor.cs). Each mode defines what can and cannot be verified.

---

## Mode Comparison Matrix

| Capability | Instrumented Binary | Release Twin | External Observation |
|:---|:---:|:---:|:---:|
| **Executable Origin** | Built with `PatchProofProbe` attached | Built from historical Git commit | Black-box historical `.exe` |
| **Probe API Access** | Full JSON-RPC over loopback | Full JSON-RPC over loopback | None (Process level only) |
| **Dynamic Cache Prep** | Supported (API controlled) | Supported (API controlled) | Filesystem manipulation only |
| **Contract Assertions** | Verified in runtime memory | Verified in runtime memory | **Cannot be verified** |
| **HTTP Request Traces** | Captured by Staging Origin | Captured by Staging Origin | Captured by Staging Origin |
| **Coverage Matrix Status** | `PASSED` / `FAILED` | `PASSED` / `FAILED` | Strictly **`UNVERIFIED`** |

---

## 1. Instrumented Binary Mode (Recommended)

### How It Works
The test executable is compiled with PatchProof's `PatchProofProbe` component present in the bootstrap scene (Execution Order `-1000`).

```csharp
// PatchProofProbe listens for Editor instructions on loopback
[DefaultExecutionOrder(-1000)]
public class PatchProofProbe : MonoBehaviour
{
    [SerializeField] private int probePort = 8898;
    // Handles Handshake, PrepareOwnedCache, InitializeAddressables, AssertContract...
}
```

### When to Use
* **Daily Development & Staging:** When creating updates for current production branches where you have access to the project source or can build a test binary.
* **Automated CI Pipelines:** Where headless runners need granular pass/fail telemetry on asset instantiation.

### Advantages
* Highest fidelity assertions.
* Can inspect instantiated `GameObject` components, memory usage, and internal Addressables operation handles.
* Directly controls cache directories via `CacheManager` APIs without OS file locking issues.

---

## 2. Release Twin Mode

### How It Works
A Release Twin is a binary built from the exact historical Git tag of your shipped release, with only the `PatchProofProbe` script added as a standalone package or script file.

```
git checkout tags/v1.0.4
# Overlay PatchProof runtime probe package
unity -batchmode -buildWindows64Player Build/Twin_v1.0.4.exe
git checkout main
```

### When to Use
* **Long-Term Support (LTS) Game Updates:** When verifying compatibility against a version shipped months ago that did not have PatchProof installed when originally compiled.

### Advantages
* Combines the pure historical code and Unity engine version of the old release with modern probe instrumentation.

---

## 3. External Observation Mode (Black Box)

### How It Works
You provide an uninstrumented, pre-existing release binary (e.g., an executable downloaded from Steam, itch.io, or an archived build folder). 

PatchProof launches the process, rewrites local DNS/hosts or utilizes command-line overrides to redirect HTTP traffic to the staging origin, and monitors network activity.

```
+-----------------------------------------------------------------------------+
| PatchProof Editor Origin (127.0.0.1:8899)                                   |
|   <-- HTTP GET /catalog_2.0.0.json (200 OK)                                 |
|   <-- HTTP GET /hero_knight.bundle (200 OK)                                 |
+-----------------------------------------------------------------------------+
                                      |
                      Wire-Level Network Telemetry Only
                                      v
+-----------------------------------------------------------------------------+
| Black-Box Historical Executable (No Probe)                                  |
|   * Downloads files successfully over wire.                                 |
|   * BUT: Did the asset actually load into memory?                           |
|   * Did a missing C# script cause a MissingComponentException?              |
|   * UNKNOWN. Internal memory cannot be verified.                            |
+-----------------------------------------------------------------------------+
```

### The Invariant: Mandatory `UNVERIFIED` Label
Because PatchProof cannot communicate with the client's internal memory or inspect operation handles in External Observation mode:
* The tile status in the Coverage Matrix is **permanently marked `UNVERIFIED`**.
* The failure category is recorded as `UnsupportedMode`.
* The failure reason explicitly states: *"External-only observation mode: internal contract load assertions cannot be verified."*

> ### Pro Tip: Honest Gating
> PatchProof refuses to mark an External Observation run as "PASSED" simply because the HTTP server logged a 200 OK response. Just because the client downloaded the bytes does not mean Unity successfully loaded the asset!
