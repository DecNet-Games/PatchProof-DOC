---
layout: default
title: Getting Started
nav_order: 2
permalink: /docs/getting-started/
has_children: false
---

# Getting Started with PatchProof
{: .no_toc }

Learn the foundational mental model of PatchProof and run your first Addressables rehearsal in under 5 minutes.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Core Concept: Rehearsal vs Unit Testing

Traditional unit tests instantiate isolated prefabs or mock catalog responses. They do not test:
1. Whether Unity's real Addressables runtime can parse a mutated catalog binary.
2. How the internal archive cache behaves when a partial download is terminated mid-stream.
3. Whether an older binary compiled two months ago can safely deserialize scripts included in a newly patched bundle.

PatchProof bridges this gap by treating releases as **first-class historical artifacts** and staging updates over a controlled, loopback network origin.

---

## The 5-Step Rehearsal Workflow

Every rehearsal in PatchProof follows a rigorous lifecycle:

```
+-----------------------------------------------------------------------------+
|                                1. REGISTER                                  |
|   Index baseline (v1.0.0) and candidate (v2.0.0) release artifacts,         |
|   catalogs, bundles, and binary executable paths into immutable manifests.  |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                                 2. DECLARE                                  |
|   Define Content Contracts declaring expected keys, asset types,            |
|   and schema invariants required for live operation.                        |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                             3. SET CONDITIONS                               |
|   Choose target client mode (Instrumented Binary), cache states             |
|   (Fresh vs WarmOld), and network fault profiles (Clean vs Latency/Drop).   |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                                 4. EXECUTE                                  |
|   Spin up local staging origin on 127.0.0.1:8899, launch the client binary,  |
|   dispatch probe commands, and execute live Addressables updates.           |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                                5. EVIDENCE                                  |
|   Analyze the Coverage Matrix, inspect HTTP request traces, verify          |
|   assertions, export Repro-Packs, and review promotion gate decisions.      |
+-----------------------------------------------------------------------------+
```

---

## 3-Minute Quick Start

### Step 1: Open the Control Room
In Unity's main menu, navigate to:
```
Tools > PatchProof > Rehearsal Room
```
The Rehearsal Room dockable window opens, showing your environment status, active release pairings, and the scenario matrix.

### Step 2: Seed the Sample Laboratory
If you are evaluating PatchProof or validating your local environment, use the built-in laboratory:
1. Navigate to `Tools > PatchProof > Local Update Lab`.
2. Click **"Generate Sample Releases & Baseline Content"**.
3. PatchProof generates two distinct release manifests in your project:
   * **v1.0.0 (Baseline):** Contains the original sample room prefab and catalog.
   * **v2.0.0 (Candidate):** Contains the updated sample room with new materials and textures.
4. Click **"Build Instrumented Windows Binary"** to create a test executable containing the runtime probe.

### Step 3: Run the Matrix
Back in the **Rehearsal Room**:
1. Select **Baseline Release:** `v1.0.0`.
2. Select **Candidate Release:** `v2.0.0`.
3. Select **Content Contract:** `SampleLab_Contract`.
4. Click **"Run Rehearsal Matrix"**.

PatchProof will:
1. Start the local HTTP origin at `http://127.0.0.1:8899/`.
2. Launch the Windows Standalone executable.
3. Prepare a **Fresh Cache** sandbox.
4. Command the runtime client to check for catalog updates, download changed bundles, and load the sample room asset.
5. Populate the **Coverage Matrix** in real time with duration, transferred bytes, and contract pass/fail status.

> ### Pro Tip: Safe Sandboxing
> PatchProof creates sandbox directories inside `Assets/../PatchProof_Runs/{runToken}/Cache/`. It never writes to or purges your Editor's personal asset cache or your user AppData directory.

---

## Next Steps
* Learn how to [Install PatchProof]({{ site.baseurl }}/docs/installation/) into your production project.
* Understand [Client Modes]({{ site.baseurl }}/docs/concepts/client-modes/) (Instrumented Binary vs Release Twin vs External Observation).
* Read the [Limitations]({{ site.baseurl }}/docs/limitations/) page for platform constraints and verified boundaries.
