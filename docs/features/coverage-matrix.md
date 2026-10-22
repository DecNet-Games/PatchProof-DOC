---
layout: default
title: Coverage Matrix Engine
parent: Features Overview
nav_order: 7
permalink: /docs/features/coverage-matrix/
---

# Coverage Matrix Engine
{: .no_toc }

Multi-dimensional scenario evaluation, partial denominators, and promotion gating truth.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Dimensions of Update Testing

An Addressables update cannot be validated by a single test run. A candidate release must be tested across multiple orthogonal axes:

$$\text{Test Space} = \text{Client Binaries} \times \text{Cache States} \times \text{Network Conditions} \times \text{Operational Intents}$$

PatchProof's **Coverage Matrix** model ([`AnalysisAndReporting.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/AnalysisAndReporting.cs)) formalizes this into an interactive, multi-tile evaluation structure.

---

## Standard Matrix Scenarios

By default, PatchProof's `RehearsalRunner` populates a 7-scenario matrix for every candidate release pairing:

| Tile | Scenario Identifier | Target Client | Initial Cache | Injected Network Fault | Primary Risk Tested |
|:---:|:---|:---:|:---:|:---:|:---|
| **1** | `FreshCache_CleanNetwork` | Baseline (v1.0.0) | Fresh (Clean) | None | Complete dependency closure on clean install |
| **2** | `WarmCache_CleanNetwork` | Baseline (v1.0.0) | WarmOld (v1.0.0) | None | Differential updates & valid cache reuse |
| **3** | `WarmCache_LatencyFault` | Baseline (v1.0.0) | WarmOld (v1.0.0) | Latency (1,500ms) | Progress bar timeouts & background downloading |
| **4** | `FreshCache_ConnectionDrop` | Baseline (v1.0.0) | Fresh (Clean) | Socket Drop (128 KB) | Mobile socket reset recovery & resumability |
| **5** | `FreshCache_TimeoutFault` | Baseline (v1.0.0) | Fresh (Clean) | Timeout (3,000ms) | HTTP header stall & reconnect dialogs |
| **6** | `MissingBundle_ContractTest` | Baseline (v1.0.0) | Fresh (Clean) | 404 Not Found | Incomplete CDN edge replication |
| **7** | `Rollback_BaselineRehearsal` | Candidate (v2.0.0) | Fresh (Clean) | None | Emergency catalog rollback safety |

---

## Tile Structure & Evidence Fields

Every tile in the matrix captures full execution telemetry:

```csharp
[Serializable]
public class CoverageTile
{
    public string scenarioName;
    public string runId;
    public ScenarioStatus status;             // Passed, Failed, Cancelled, Unverified
    public FailureCategory failureCategory;   // None, NetworkFaultSimulation, ContractViolation, etc.
    public string failureReason;              // Human-readable diagnostic message
    public double durationSeconds;            // Wall-clock execution time
    public long wireBytesTransferred;         // Bytes streamed from staging origin
    public long decompressedDiskBytes;        // Physical cache footprint
    public List<ClientAssertion> assertions;  // Results of ContentContract key assertions
    public List<HttpTrace> requestTraces;     // Full HTTP request timeline
    public List<ProbeEvent> probeEvents;      // Internal client probe console logs
}
```

---

## The Promotion Gating Logic

The Coverage Matrix is not merely an informational dashboard; it is a **strict gate for continuous integration**.

Implemented in `CoverageMatrix.cs`:
```csharp
public bool CanPromoteToProduction()
{
    // A release can ONLY be promoted if every tile executed and passed!
    if (tiles == null || tiles.Count == 0) return false;
    return tiles.All(t => t.status == ScenarioStatus.Passed);
}
```

### Exit Code Truth
When executed via the command line or CI runners:
* **Exit Code `0`:** All matrix tiles executed with verified runtime probe memory assertions and passed 100%.
* **Exit Code `1`:** One or more tiles failed (contract broken, crash, or unhandled exception).
* **Exit Code `2`:** Configuration error (manifest invalid, executable missing, port locked).
* **Exit Code `3`:** Matrix contains **Unverified Cells** (e.g., run in External Observation mode, or a fault rule failed to trigger).

> ### Pro Tip: No Partial Credit
> In PatchProof, an exit code of `3` is treated as a build blocker by CI orchestrators (GitHub Actions, GitLab CI, TeamCity). It tells your team: *"The update didn't crash, but you didn't prove it worked either."*
