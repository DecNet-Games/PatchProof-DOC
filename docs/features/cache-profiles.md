---
layout: default
title: Cache Profiles & Isolation
parent: Features Overview
nav_order: 6
permalink: /docs/features/cache-profiles/
---

# Cache Profiles & Process Isolation
{: .no_toc }

Technical implementation of cache directory sandboxing, run tokens, and concurrent process management.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Architectural Imperative: No Cross-Run Contamination

When running rehearsals back-to-back, a critical testing pitfall is **cross-run contamination**:
* Test Run A downloads a candidate bundle.
* Test Run B starts, expecting a clean environment, but finds the cached bundle from Run A, falsely reporting a "cache hit" or bypassing the network update altogether.
* An engineer manually deletes files while a background player is running, causing file-locking crashes.

PatchProof solves this with dual isolation engines: [`CacheManager.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/Orchestration/CacheManager.cs) and [`ProcessManager.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/Orchestration/ProcessManager.cs).

---

## Sandboxed Cache Topologies

For every scenario, PatchProof generates a cryptographically unique run token (e.g., `run_7b82f91a`). All cache operations are restricted to this folder:

```
{ProjectRoot}/PatchProof_Runs/
└── run_7b82f91a/
    ├── Cache/
    │   ├── com.unity.addressables/     # Downloaded catalogs & primary hash files
    │   └── UnityCache/Shared/          # Serialized .bundle archives
    ├── Traces/
    │   └── requests.json               # Recorded HTTP request logs
    └── Logs/
        └── client_player.log           # Standalone player output log
```

### Path Traversal Guard
To protect developer systems, `CacheManager` enforces strict path sanitization:
```csharp
if (runToken.Contains("..") || runToken.Contains("/") || runToken.Contains("\\"))
{
    throw new ArgumentException("Run token contains illegal path traversal characters.");
}
```

---

## Process Lifetime & Safe Termination

Running standalone player executables in automated pipelines requires bulletproof process management.

### The Owned Process Registry
When PatchProof launches a player executable, it tracks the operating system Process ID (PID) alongside the assigned `runToken` in an internal dictionary:
```csharp
private static readonly Dictionary<string, Process> _ownedProcesses = new Dictionary<string, Process>();
```

### Preventing Rogue Termination of System Processes
Before terminating any process, `ProcessManager.StopOwnedProcess(runToken)` validates ownership:
* If the `runToken` is not in the owned registry, the call returns `false` and exits immediately.
* It will **never terminate an unowned PID** or system process.

```csharp
public static bool StopOwnedProcess(string runToken)
{
    if (string.IsNullOrEmpty(runToken) || !_ownedProcesses.TryGetValue(runToken, out var proc))
    {
        return false; // Safely ignore unowned requests
    }
    // Graceful close followed by hard kill if unresponsive
}
```

### Interruption Recovery
If the Unity Editor crashes, restarts, or triggers a domain reload while a rehearsal is running:
1. `ProcessManager` detects orphaned child processes on next domain startup.
2. It cross-references active PIDs against the cached run token file.
3. It cleanly terminates abandoned instances and purges their transient run directories.

---

## Cache Assessment API

Engineers can query the state of any run directory using `CacheManager.AssessState(directoryPath)`:

| Assessed State | Directory Condition |
|:---|:---|
| `CacheState.Fresh` | Folder does not exist, or contains 0 bytes. |
| `CacheState.WarmOld` | Contains valid baseline catalog hash and matching baseline bundles. |
| `CacheState.Mixed` | Contains candidate bundles mixed with stale or incomplete temporary files. |
| `CacheState.Corrupt` | Contains damaged file headers that fail archive magic number validation. |

> ### Pro Tip: Inspecting Disk Sizes
> In the Rehearsal Room details pane, PatchProof displays both **Downloaded Wire Bytes** (network transfer) and **Decompressed Disk Footprint** (physical cache size). Use this to catch unexpected bundle inflation before mobile rollouts!
