---
layout: default
title: Lab Walkthrough
parent: Sample Laboratory
nav_order: 1
permalink: /docs/sample/sample-laboratory/
---

# Sample Laboratory Walkthrough
{: .no_toc }

Hands-on walkthrough of the bundled update laboratory: Room v1, Valid v2, and Broken v2.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Laboratory Overview

The **Sample Update Laboratory** (`Tools > PatchProof > Local Update Lab`) demonstrates how PatchProof catches subtle update regressions before they hit players.

The lab defines three release snapshots:
1. **v1.0.0 (Baseline):** Ships `SampleRoom_v1` with basic wooden textures and a `SampleRoomEntity` component tracking room lighting and volume.
2. **v2.0.0 (Valid Candidate):** Updates `SampleRoom` with neon cyber textures, high-resolution models, and additional sound effects while preserving all C# component bindings.
3. **v2.0.0 (Broken Candidate):** Deliberately seeds common real-world shipping errors:
   * Key `"Prefabs/SampleRoom"` has been altered to point to a deleted script.
   * Key `"Audio/AmbientLoop"` was deleted without being declared in `permittedRemovals`.
   * Shader variant stripping removed a required variant on the floor material.

---

## Step 1: Generating Lab Releases

1. In Unity, open `Tools > PatchProof > Local Update Lab`.
2. Click the primary button: **"Generate Sample Content & Releases"**.
3. PatchProof automatically:
   * Configures Addressables group settings for the sample assets.
   * Compiles the Addressables bundles for `v1.0.0`.
   * Copies and modifies the assets to build `v2.0.0`.
   * Registers both manifests into the **Release Registry**.
   * Creates the `SampleLab_Contract` Content Contract.

---

## Step 2: Compiling the Test Binary

1. In the **Local Update Lab** window, navigate to the **Binary Builder** tab.
2. Ensure **Target Platform** is set to **Windows Standalone (x86_64)**.
3. Click **"Build Instrumented Windows Binary"**.
4. Unity builds the executable to `Builds/PatchProof_SampleClient.exe`. The build automatically embeds `PatchProofProbe` in the initial scene.

---

## Step 3: Running the Valid Candidate Rehearsal

1. Open `Tools > PatchProof > Rehearsal Room`.
2. Select **Baseline Release:** `SampleLab_v1.0.0`.
3. Select **Candidate Release:** `SampleLab_v2.0.0_Valid`.
4. Select **Contract:** `SampleLab_Contract`.
5. Click **"Run Rehearsal Matrix"**.
6. **Result:** All 7 tiles complete with status **PASSED** (Green).

---

## Step 4: Running the Broken Candidate Rehearsal

Now, observe what happens when a release contains breaking bugs:
1. In the **Rehearsal Room**, change **Candidate Release** to `SampleLab_v2.0.0_Broken`.
2. Click **"Run Rehearsal Matrix"**.
3. **Result:**
   * **Tile 1 (Fresh Cache):** **FAILED** (`ContractViolation`). Diagnostic reason: *"Required key 'Audio/AmbientLoop' was not found in candidate catalog."*
   * **Tile 2 (Warm Old Cache):** **FAILED** (`MissingScriptComponent`). Diagnostic reason: *"Prefab 'SampleRoom_v2_Broken' has a missing MonoBehaviour component on child 'LightRig'."*

### Investigating with Repro-Pack
1. Select the failed **Tile 2**.
2. Click **"Export Repro-Pack"**.
3. Unzip the resulting archive and open `client_log.txt` to see the exact Unity stack trace caught by the probe!

> ### Pro Tip: Clean Reset
> Want to reset the laboratory to its pristine initial state? Click **"Purge Lab Content & Temp Builds"** in the Local Lab window. PatchProof deletes only sample-related assets, leaving your project untouched.
