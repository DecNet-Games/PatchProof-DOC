---
layout: default
title: Repro-Packs
parent: Features Overview
nav_order: 11
permalink: /docs/features/repro-packs/
---

# Repro-Packs & Failure Reproduction
{: .no_toc }

Exporting self-contained, reproducible failure archives with exact catalogs, faults, and replay steps.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The "Works On My Machine" Trap

When a continuous integration pipeline flags an Addressables update failure, debugging it is notoriously tedious:
* The CI runner had specific bundles in a temporary cache that the local developer does not have.
* The failure only happens when downloading over a high-latency connection with an abrupt socket drop at byte 131,072.
* Developers spend hours attempting to recreate the exact branch state, build settings, and network conditions.

PatchProof's **Repro-Pack Engine** ([`ReproPack.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/RunRecords.cs)) eliminates this friction by generating **self-contained bug reproduction packages with a single click**.

---

## Anatomy of a Repro-Pack Archive

A Repro-Pack is a compressed `.zip` archive containing everything required to reproduce a failure in a clean workspace:

```
repro_run_8f91c2a0_FreshCache_Drop.zip
├── manifest.json              # ReproPack metadata (timestamps, versions, platform)
├── catalogs/
│   ├── baseline_catalog.json  # Exact baseline catalog snapshot
│   └── candidate_catalog.json # Exact candidate catalog snapshot
├── fault_profile.json         # Exact FaultProfile active during the failure
├── request_traces.json        # Recorded HTTP telemetry showing wire activity
├── client_log.txt             # Standalone player stdout/stderr log
└── replay_instructions.md     # Step-by-step instructions for one-click replay
```

---

## One-Click Export from the Rehearsal Room

When any scenario tile in the Coverage Matrix fails or throws an unhandled exception:

1. Select the failed tile in the **Rehearsal Room** grid.
2. In the tile details pane, click **"Export Repro-Pack"**.
3. Choose an output destination (e.g., `Desktop/repro_drop_failure.zip`).
4. PatchProof gathers the active catalogs, fault rules, traces, and client logs, and packages them into a clean zip archive.
5. Attach the Repro-Pack to your JIRA, GitHub Issue, or Slack message.

---

## Replaying a Seeded Failure

Any engineer on your team can reproduce the exact failure without rebuilding the project:

### Step 1: Import the Repro-Pack
1. Open Unity with PatchProof installed.
2. Navigate to `Tools > PatchProof > Rehearsal Room`.
3. Click **"Import & Replay Repro-Pack"** in the top utility toolbar.
4. Select the `.zip` archive.

### Step 2: Automatic Test Reconstruction
PatchProof automatically:
1. Re-constructs the exact virtual staging routes on `127.0.0.1:8899`.
2. Restores the exact `FaultProfile` (including byte offsets and corruption thresholds).
3. Configures the exact initial cache state (`Fresh`, `WarmOld`, or `MixedPartial`).
4. Launches the local standalone player executable.

### Step 3: Attach the Debugger
Because the environment is 100% deterministic and runs locally:
1. Attach your IDE debugger (Visual Studio, Rider, or VS Code) to the running player process.
2. Step through your Addressables download callbacks, error handlers, and state machines as the fault fires.
3. Fix the bug, rebuild the binary, and re-run the matrix to verify the fix!

> ### Pro Tip: CI Artifact Archiving
> In headless CI configurations, configure your build runner to automatically export and archive Repro-Packs whenever `PatchProofHeadlessRunner` exits with a non-zero code. Developers never have to rebuild failing CI states locally!
