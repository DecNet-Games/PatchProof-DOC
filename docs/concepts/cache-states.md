---
layout: default
title: Cache States & Invariants
parent: Core Concepts
nav_order: 4
permalink: /docs/concepts/cache-states/
---

# Cache States & Invariants
{: .no_toc }

Deep dive into how PatchProof creates reproducible client storage topologies and isolates concurrent runs.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Role of Local Cache in Update Failures

When Unity's Addressables system evaluates a content update, it queries local storage before making network requests:
1. It compares the remote catalog hash with the local cached hash in `Application.persistentDataPath/com.unity.addressables/`.
2. For each required asset bundle, it checks if the bundle's internal hash matches an existing `.bundle` file in `UnityEngine.Caching`.
3. If a hash mismatch exists, or if a partially downloaded archive was corrupted during an aborted session, Unity may crash, fail silently, or continuously re-download gigabytes of data.

To test these edge cases reliably, PatchProof provides deterministic **Cache Profiles**.

---

## Supported Cache States

```
+-----------------------------------------------------------------------------+
|                               Cache Profiles                                |
+-----------------------------------------------------------------------------+
|   1. Fresh Cache          | Zero files; clean install state.                |
|   2. Warm Old Cache       | Baseline bundles seeded; tests differential.    |
|   3. Mixed / Partial Cache| Partial files & invalid hashes; tests recovery. |
+-----------------------------------------------------------------------------+
```

### 1. Fresh Cache (`CacheState.Fresh`)
* **Physical Topology:** Completely empty directory.
* **Simulates:** A new player installing the game for the first time, or a player who clicked "Clear App Storage" in system settings.
* **Verification Objective:** Confirms that the candidate release contains complete, self-contained dependency closures. If a bundle depends on an unlisted base asset, a fresh install will fail immediately.

### 2. Warm Old Cache (`CacheState.WarmOld`)
* **Physical Topology:** Pre-populated with catalog files, hash files, and asset bundles from the registered **Baseline Release (v1.0.0)**.
* **Simulates:** An active player who has been playing the game on yesterday's release and launches the app when the new patch goes live.
* **Verification Objective:**
  * Asserts that unchanged asset bundles are successfully reused from disk without triggering redownloads over the network.
  * Verifies that modified bundles correctly invalidate their old cached counterparts without file-locking crashes.

### 3. Mixed / Partial Cache (`CacheState.Mixed`)
* **Physical Topology:** Contains valid baseline bundles alongside truncated (`.temp`) files and corrupted byte streams.
* **Simulates:** A player whose network connection was severed while downloading an earlier patch, leaving orphaned partial files on the filesystem.
* **Verification Objective:** Confirms that the Addressables cache initialization sequence detects corrupt headers, purges the damaged blocks, and cleanly re-fetches the asset.

---

## Run-Token Isolation Invariants

When running automated test suites or parallel CI jobs, multiple test runs must never overwrite each other's cache directories.

PatchProof enforces strict folder isolation in [`CacheManager.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/Orchestration/CacheManager.cs):

```csharp
public static string GetOwnedRunCacheDirectory(string runToken)
{
    // Every run gets a cryptographically unique directory
    string cleanToken = SanitizeToken(runToken);
    return Path.Combine(BaseRunsDirectory, cleanToken, "Cache");
}
```

### Path Traversal Protection
PatchProof rejects any run token containing path traversal characters (`..`, `/`, `\`) or illegal characters. Attempts to escape the sandbox throw an `ArgumentException` and fail the rehearsal immediately.

### Safe Cleanup Guarantees
When a rehearsal finishes (or is cancelled), PatchProof invokes `CacheManager.ClearOwnedRunCache(runToken)`:
* Only the subfolder matching the specific `runToken` is removed.
* Global Unity caches, Editor caches, and unrelated run tokens are preserved.
* If a file is locked by a lingering process, PatchProof's `ProcessManager` terminates the owned child PID before attempting directory deletion.
