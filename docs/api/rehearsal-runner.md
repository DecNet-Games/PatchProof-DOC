---
layout: default
title: RehearsalRunner API
parent: API Reference
nav_order: 1
permalink: /docs/api/rehearsal-runner/
---

# `RehearsalRunner` Class
{: .no_toc }

Namespace: `Decnet.PatchProof.Editor`  
Assembly: `Decnet.PatchProof.Editor.dll`
{: .lead }

The `RehearsalRunner` class is the central orchestrator responsible for initializing the staging origin, launching standalone client processes, dispatching scenario tiles, and collecting evidence.

---

## Public Events

```csharp
public event Action<string, float> OnProgress;
public event Action<CoverageTile> OnTileCompleted;
```

* **`OnProgress`:** Fired when a scenario begins or updates its status. Passes a human-readable description and normalized float progress `[0.0, 1.0]`.
* **`OnTileCompleted`:** Fired immediately after an individual `CoverageTile` concludes execution and evidence is recorded.

---

## Public Properties

```csharp
public bool IsCancelled { get; }
```
Returns `true` if a cancellation request has been signaled via `Cancel()`.

---

## Public Methods

### `RunMatrixAsync`
Executes a complete multi-scenario matrix asynchronously.

```csharp
public async Task<CoverageMatrix> RunMatrixAsync(
    ReleaseManifest baselineRelease,
    ReleaseManifest candidateRelease,
    ContentContract contract,
    EndpointProfile endpoint,
    List<CacheProfile> cacheProfiles,
    List<FaultProfile> faultProfiles,
    ClientMode clientMode,
    string contentDirectory
);
```

#### Parameters
* **`baselineRelease`:** Registered baseline release manifest (e.g., currently live production build).
* **`candidateRelease`:** Registered candidate release manifest under evaluation.
* **`contract`:** Active content contract defining expected keys and types.
* **`endpoint`:** Staging origin configuration (port, bind IP, timeout). Pass `null` for defaults (`127.0.0.1:8899`).
* **`cacheProfiles`:** List of cache state configurations to evaluate.
* **`faultProfiles`:** List of network fault configurations to inject.
* **`clientMode`:** Execution mode (`InstrumentedBinary`, `ReleaseTwin`, or `ExternalObservation`).
* **`contentDirectory`:** Path to folder containing candidate bundle archives.

#### Returns
* `Task<CoverageMatrix>`: Completed coverage matrix containing populated tiles, request traces, and assertions.

---

### `Cancel`
Signals graceful cancellation of the active rehearsal run.

```csharp
public void Cancel();
```

* Flags `IsCancelled = true`.
* Aborts pending HTTP transactions on the local staging origin.
* Marks current and remaining scenario tiles with `ScenarioStatus.Cancelled`.
* Preserves all completed partial evidence prior to cancellation.
