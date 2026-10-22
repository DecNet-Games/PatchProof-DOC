---
layout: default
title: The Rehearsal Model
parent: Core Concepts
nav_order: 1
permalink: /docs/concepts/rehearsal-model/
---

# The Rehearsal Model
{: .no_toc }

A complete breakdown of the 5-stage lifecycle that powers PatchProof release verification.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Rehearsals Matter

In live game operations, shipping an Addressables patch is not simply copying files to an S3 or Google Cloud Storage bucket. It is a live protocol migration:

1. **Old Code Meets New Data:** Older installed binaries run code compiled months ago against newly serialized asset bundles.
2. **Persistent Caches:** Previous versions leave behind cached catalogs, hash files, and asset bundles in `Application.persistentDataPath`.
3. **Unpredictable Networks:** Real-world networks drop connections, inject HTTP proxies, or throttle transfers.

The **PatchProof Rehearsal Model** replicates these exact variables deterministically on your local development workstation or CI runner.

---

## The 5 Stages in Detail

```
+---------------+     +---------------+     +--------------------+
|  1. Register  | --> |  2. Declare   | --> | 3. Set Conditions  |
+---------------+     +---------------+     +--------------------+
                                                      |
                      +---------------+               v
                      |  5. Evidence  | <--- +--------------------+
                      +---------------+      |    4. Execute      |
                                             +--------------------+
```

### Stage 1: Register (Immutable Manifests)
Before any code runs, both releases are registered into immutable data structures:
* **Release ID:** Semantic version or commit hash (e.g., `v1.0.4-rc2`).
* **Catalog Snapshots:** Path to the built `catalog.json` or `catalog.bin`, including its primary hash.
* **Artifact Manifest:** A cryptographically signed dictionary mapping relative file paths to their exact file size and **SHA-256** checksum.
* **Client Descriptor:** Path to the compiled Standalone binary representing that release.

If any bundle is modified on disk after registration, PatchProof detects the hash divergence and rejects the rehearsal to avoid invalid evidence.

### Stage 2: Declare (Content Contracts)
A Content Contract formalizes the expected runtime contract between the game code and the Addressables catalog. It declares:
* **Required Keys:** Primary keys and labels that must resolve.
* **Expected Types:** Exact C# `Type` names (e.g., `UnityEngine.AudioClip`, `MyGame.Characters.HeroData`).
* **Sub-Object Invariants:** Specific component types that must be attached to loaded prefabs.
* **Permitted Removals:** Explicitly acknowledged keys that were deprecated or deleted in the candidate release.

### Stage 3: Set Conditions (Scenarios & Sandboxes)
A scenario is the intersection of:
1. **Target Client:** Historical binary (v1.0.0) or candidate twin (v2.0.0).
2. **Cache State:**
   * `Fresh`: Zero cached files; pristine clean-install state.
   * `WarmOld`: Pre-seeded with baseline release bundles and catalog.
   * `MixedPartial`: Pre-seeded with aborted, zero-byte, or partial downloads.
3. **Network Fault Profile:**
   * `Clean`: Zero latency, direct localhost delivery.
   * `Latency`: Configurable artificial delay per chunk.
   * `Abrupt Drop`: Connection terminated at offset $B$.
   * `HTTP Fault`: Injected 404/500 response codes.
   * `Corruption`: Flipped byte ranges in bundle payloads.

### Stage 4: Execute (Loopback Staging & Probe Dispatch)
Execution takes place completely offline over loopback:
1. **Local Staging Origin:** Starts an internal HTTP server at `http://127.0.0.1:{port}/`. It mounts the candidate release directory to virtual routes.
2. **Client Process Launch:** PatchProof's `ProcessManager` spawns the standalone player with a unique `--patchproof-token {runToken}`.
3. **Cache Sandboxing:** The client isolates its Addressables cache to a subfolder specific to that token.
4. **Command Protocol:** The Editor sends structured JSON-RPC commands to the client's internal probe via HTTP port `8898`:
   * `Handshake`
   * `PrepareOwnedCache`
   * `InitializeAddressables`
   * `CheckForUpdate`
   * `DownloadContent`
   * `AssertContract`

### Stage 5: Evidence (Audit Trails & Reports)
The rehearsal concludes by aggregating all telemetry:
* **Matrix Tile Status:** `PASSED`, `FAILED`, `CANCELLED`, or `UNVERIFIED`.
* **Telemetry Traces:** Exact HTTP requests, byte counts, and response latency.
* **Contract Assertions:** Pass/fail logs for every declared asset key.
* **Artifact Export:** Self-contained HTML promotion reports and JSON CI outputs.
