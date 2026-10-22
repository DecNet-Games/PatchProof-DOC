---
layout: default
title: First Rehearsal Walkthrough
parent: Workflows
nav_order: 1
permalink: /docs/workflows/first-rehearsal/
---

# First Rehearsal Walkthrough
{: .no_toc }

A complete operational guide for taking your project from unverified builds to your first green rehearsal matrix.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

Before starting your first rehearsal, ensure:
1. PatchProof is imported into your Unity project (`Assets/Decnet/PatchProof`).
2. The Addressables package is active and your content is grouped under `Window > Asset Management > Addressables > Groups`.
3. You have built your Addressables content at least once (`Build > New Build > Default Build Script`).

---

## Step 1: Open the Guided Setup Wizard

Navigate to:
```
Tools > PatchProof > Guided Setup Wizard
```

The wizard checks your project settings:
* **Addressables Settings Asset:** Confirms `AddressableAssetSettings.asset` is found in `Assets/AddressableAssetsData/`.
* **Build Remote Catalog:** Verifies that "Build Remote Catalog" is checked in your Addressables settings.
* **Remote Paths:** Ensures your remote load path is set to a variable URL or default staging template.

Click **"Apply Guided Configuration"** to establish initial registry directories.

---

## Step 2: Build and Register Baseline Release (v1.0.0)

1. Open `Window > Asset Management > Addressables > Groups`.
2. Click **Build > New Build > Default Build Script**.
3. Once the build finishes, build a Windows Standalone executable:
   * `File > Build Profiles` (or `Build Settings`).
   * Target: **Windows Standalone (x86_64)**.
   * Path: `Builds/Game_v1.0.0/Game.exe`.
   * Ensure `PatchProofProbe` is attached in your starting scene.
4. Open `Tools > PatchProof > Release Registry`.
5. Click **"Register Release"**:
   * **Release ID:** `v1.0.0`
   * **Platform:** `WindowsStandalone`
   * **Content Folder:** Point to `ServerData/StandaloneWindows64/`.
   * **Executable:** Point to `Builds/Game_v1.0.0/Game.exe`.
6. Click **"Lock Manifest & Compute Checksums"**.

---

## Step 3: Create Content Modifications for Candidate (v2.0.0)

Now, simulate a realistic content update in your project:
1. Modify a material or add a new texture to an addressable prefab.
2. In the Addressables Groups window, click **Build > Update a Previous Build**.
3. Select the `addressables_content_state.bin` generated during the v1.0.0 build.
4. Addressables builds the updated catalog and differential bundles!
5. Open `Tools > PatchProof > Release Registry`.
6. Register the candidate release:
   * **Release ID:** `v2.0.0`
   * **Content Folder:** Point to the newly updated build folder.
   * **Executable:** Point to the existing `v1.0.0` executable (simulating an older client running against the new update!).
7. Click **"Lock Manifest & Compute Checksums"**.

---

## Step 4: Define Your Content Contract

1. In the **Release Registry**, switch to the **Content Contracts** tab.
2. Click **"New Contract"** and name it `CoreReleaseContract`.
3. Add the primary asset keys that your game loads on startup.
4. Click **"Validate Contract Offline"** to verify that the candidate catalog contains these keys.

---

## Step 5: Execute in the Rehearsal Room

1. Navigate to `Tools > PatchProof > Rehearsal Room`.
2. Set:
   * **Baseline:** `v1.0.0`
   * **Candidate:** `v2.0.0`
   * **Contract:** `CoreReleaseContract`
   * **Client Mode:** `Instrumented Binary`
3. Click **"Run Full Rehearsal Matrix"**.
4. Observe the console output as the local origin initializes, the executable spawns, the probe completes the handshake, and the scenario tiles populate with green checkmarks.
5. Click **"Export Release Report"** to save your first verified release dossier!
