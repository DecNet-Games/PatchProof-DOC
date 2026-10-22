---
layout: default
title: Rehearsal Room
parent: Features Overview
nav_order: 1
permalink: /docs/features/rehearsal-room/
---

# Rehearsal Room Control Window
{: .no_toc }

The central operational console for orchestrating, visualizing, and auditing release scenarios.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Window Overview

The **Rehearsal Room** (`Tools > PatchProof > Rehearsal Room`) is a dockable Unity Editor window implemented in [`RehearsalRoomWindow.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/UI/RehearsalRoomWindow.cs). It acts as mission control for preparing and executing release tests.

```
+-----------------------------------------------------------------------------+
| Rehearsal Room [Unity 6000.5.4f1 LTS | Addressables 2.9.1 | Win64]           |
+-----------------------------------------------------------------------------+
| Baseline: [ v1.0.0 (Win64) v ]  Candidate: [ v2.0.0 (Win64) v ]             |
| Contract: [ CoreGameContract v ] Client Mode: [ InstrumentedBinary v ]      |
+-----------------------------------------------------------------------------+
| [ RUN FULL REHEARSAL MATRIX ]   [ CANCEL RUN ]   [ EXPORT REPORT ]          |
+-----------------------------------------------------------------------------+
| Progress: Executing scenario 3/7: FreshCache_TimeoutFault [=========>     ] |
+-----------------------------------------------------------------------------+
| COVERAGE MATRIX:                                                            |
| [1. Fresh / Clean: PASSED (1.2s)]   [2. Warm / Clean: PASSED (0.4s)]        |
| [3. Fresh / Latency: PASSED (3.1s)] [4. Fresh / Drop: FAILED (0.8s)]        |
| [5. Fresh / Timeout: PASSED (3.0s)] [6. Missing Bundle: PASSED (0.2s)]      |
| [7. Rollback: PASSED (0.5s)]                                                |
+-----------------------------------------------------------------------------+
| SELECTED TILE DETAILS (Tile 4: Fresh / Drop):                               |
| Run ID: run_a1b2c3d4 | Duration: 0.82s | Transferred: 131,072 bytes         |
| Status: FAILED | Category: NetworkFaultSimulation                           |
| Reason: Socket aborted at byte offset 131072 during hero_knight.bundle fetch|
| [ View HTTP Request Traces ]   [ Export Repro-Pack ]                        |
+-----------------------------------------------------------------------------+
```

---

## Key Interface Sections

### 1. Environment Banner
At the top of the window, PatchProof inspects and displays your active environment:
* **Active Unity Version:** Must match your target LTS runtime.
* **Addressables Version:** Verified against the installed package cache (`2.9.1`).
* **Active Platform Target:** Highlights current build target (e.g., `WindowsStandalone64`).

### 2. Configuration Selectors
* **Baseline Release:** The historical version to test against (e.g., currently live production version).
* **Candidate Release:** The newly built candidate update.
* **Content Contract:** Active contract rules enforcing required asset keys and schemas.
* **Client Execution Mode:** Choose between `InstrumentedBinary`, `ReleaseTwin`, or `ExternalObservation`.

### 3. Execution Controls
* **Run Full Rehearsal Matrix:** Spawns the local origin, launches the client binary, and executes all defined scenarios sequentially.
* **Cancel Run:** Instantly signals cancellation via `RehearsalRunner.Cancel()`. Gracefully stops child processes, stops the HTTP server, marks remaining tiles as `CANCELLED`, and preserves partial evidence.
* **Export Report:** Compiles the current matrix into standalone HTML and JSON release reports.

### 4. Interactive Matrix Grid
Every tile in the scenario matrix is rendered with real-time status badges:
* **Green:** `PASSED` with execution duration.
* **Red:** `FAILED` with failure category.
* **Amber:** `UNVERIFIED` (indicates gaps in evidence).
* **Gray:** `PENDING` or `CANCELLED`.

Clicking any tile opens the **Selected Tile Details** pane, providing immediate access to request traces, console events, and one-click **Repro-Pack** generation.

> ### Pro Tip: Layout Docking
> Dock the Rehearsal Room next to your Game View and Console. During instrumented binary runs, you can observe both the Editor matrix and the running standalone client simultaneously.
