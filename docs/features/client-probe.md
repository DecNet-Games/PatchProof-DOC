---
layout: default
title: Runtime Client Probe
parent: Features Overview
nav_order: 5
permalink: /docs/features/client-probe/
---

# Runtime Client Probe
{: .no_toc }

Lightweight runtime fixture, loopback command protocol, and in-memory contract validation.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why an In-Binary Probe?

External tools can tell you that a process launched and made network requests. They cannot tell you:
* Did `Addressables.InitializeAsync()` succeed?
* Did `Addressables.UpdateCatalogs()` find and register the new catalog?
* Did `Addressables.LoadAssetAsync<GameObject>("Prefabs/Hero")` instantiate without missing component scripts?
* How many megabytes of RAM were allocated by the updated bundle?

PatchProof's **Runtime Probe** ([`PatchProofProbe.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/RuntimeProbe/PatchProofProbe.cs)) embeds directly into standalone test binaries to answer these questions with zero ambiguity.

---

## Architectural Lifecycle

The probe is a `MonoBehaviour` assigned an execution order of `-1000` to ensure it wakes up before any game logic executes:

```csharp
[DefaultExecutionOrder(-1000)]
public class PatchProofProbe : MonoBehaviour
{
    public static PatchProofProbe Instance { get; private set; }
    [SerializeField] private int probePort = 8898;
    // ...
}
```

```
+-----------------------------------------------------------------------------+
|                     Standalone Player Process (Client)                      |
+-----------------------------------------------------------------------------+
|  Awake() -> Parse CommandLine (--patchproof-port, --patchproof-token)        |
|  Start HTTP Server on http://127.0.0.1:8898/                                |
|  Listen for incoming JSON-RPC command requests from Editor                   |
|  Dispatch commands to Unity Main Thread via EnqueueMainThread()              |
|  Execute Addressables operations & assert Content Contracts                 |
|  Return ProbeResponse JSON with full event log & client assertions           |
+-----------------------------------------------------------------------------+
```

---

## The Probe Protocol (`ProbeCommandType`)

Communication between the Editor and the client binary uses strongly-typed JSON requests defined in [`ProbeProtocol.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/RuntimeProbe/ProbeProtocol.cs):

### 1. `ProbeCommandType.Handshake`
* Verifies connectivity between Editor and client.
* Returns engine version, Addressables version, operating platform, and current memory footprint.

### 2. `ProbeCommandType.PrepareOwnedCache`
* Takes `desiredCacheState` (`Fresh`, `WarmOld`, `Mixed`).
* Prepares the isolated cache sandbox on the client's local drive without requiring OS file permissions outside the sandbox.

### 3. `ProbeCommandType.InitializeAddressables`
* Dynamically overrides the internal Addressables remote load paths to point to the local staging origin (`http://127.0.0.1:8899/`).
* Calls `Addressables.InitializeAsync()`.

### 4. `ProbeCommandType.CheckForUpdate`
* Calls `Addressables.CheckForCatalogUpdates()`.
* Returns the list of modified catalogs discovered on the staging origin.

### 5. `ProbeCommandType.UpdateCatalogs`
* Calls `Addressables.UpdateCatalogs()`.
* Activates new catalog instances into the active Addressables locator list.

### 6. `ProbeCommandType.AssertContract`
* Accepts a serialized `ContentContract` JSON string.
* Iterates through every declared key, loads the asset asynchronously, inspects `loadedObject.GetType()`, and validates required component schemas.
* Releases the loaded asset handles to clean up memory.

---

## Embedding the Probe in Your Project

### Automatic Inclusion (One-Click)
When you build an instrumented player from `Tools > PatchProof > Local Update Lab`:
1. PatchProof automatically instantiates the probe prefab into the initial scene.
2. It configures default ports and command-line parsing.
3. It compiles the binary with developer-mode symbols.

### Manual Inclusion (Existing Projects)
1. In your project's first loading scene (Bootstrap / Splash), create an empty GameObject named `[PatchProof_Probe]`.
2. Attach the `PatchProofProbe` component.
3. Ensure **Auto Start Server** is checked.
4. Ensure the script is wrapped in `#if DEVELOPMENT_BUILD || UNITY_EDITOR` (PatchProof automatically wraps server startup to prevent probe code from running in final public production builds).

> ### Pro Tip: Security Disclaimers
> The probe's HTTP listener is bound strictly to `127.0.0.1` and requires a matching `--patchproof-token` argument passed at process launch. Unauthenticated network requests are rejected with HTTP 403 Forbidden.
