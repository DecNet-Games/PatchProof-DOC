---
layout: default
title: Quick Start Guide
nav_order: 4
permalink: /docs/quick-start/
has_children: false
---

# Quick Start Guide
{: .no_toc }

A practical, step-by-step walkthrough of creating your first release manifest, declaring a contract, and executing an update rehearsal.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario Overview

In this guide, we will rehearse updating an imaginary game from **Release 1.0.0** to **Release 2.0.0**:
* **Release 1.0.0:** Shipped with a main menu asset named `MainMenu_v1`.
* **Release 2.0.0:** Modifies `MainMenu` with high-resolution textures and introduces a new character prefab `Hero_Knight`.
* **Goal:** Verify that a standalone player installed with 1.0.0 can cleanly download the 2.0.0 update over a simulated high-latency connection without crashing or corrupting its cache.

---

## Step 1: Register Release Manifests

Release Manifests freeze a release's metadata, catalog path, and bundle hashes.

1. Open `Tools > PatchProof > Release Registry`.
2. Click **"New Release Manifest"**.
3. Fill in the details for **v1.0.0**:
   * **Release ID:** `v1.0.0`
   * **Platform:** `WindowsStandalone`
   * **Catalog Path:** Select the build output catalog file (`catalog_1.0.0.json` or `.bin`).
   * **Content Folder:** Select the folder holding the 1.0.0 bundles.
   * **Executable Path:** Path to your compiled 1.0.0 Windows executable (`Game_v1.0.0.exe`).
4. Click **"Save & Hash Manifest"**. PatchProof computes cryptographic SHA-256 digests for all referenced files to ensure immutability.
5. Repeat the process to create the **v2.0.0 (Candidate)** manifest pointing to your newly built candidate bundles.

---

## Step 2: Define a Content Contract

Content Contracts ensure that critical assets exist in the candidate release and instantiate correctly.

1. In the **Release Registry**, switch to the **Content Contracts** tab.
2. Click **"Create Contract"**.
3. Name it `CoreGameContract`.
4. Add target assets:
   * **Key:** `Prefabs/MainMenu` &rarr; **Expected Type:** `UnityEngine.GameObject`
   * **Key:** `Prefabs/Hero_Knight` &rarr; **Expected Type:** `UnityEngine.GameObject`
   * **Key:** `Audio/MenuTheme` &rarr; **Expected Type:** `UnityEngine.AudioClip`
5. Click **"Validate Contract Against Candidate"**. PatchProof parses the candidate catalog offline to confirm that all declared keys are present.

---

## Step 3: Configure Network Faults (Optional)

To test how your game recovers from a network interruption:

1. Open `Tools > PatchProof > Origin & Fault Profiles`.
2. Click **"New Fault Profile"**.
3. Name the profile `MobileSubwayDropout`.
4. Add a rule:
   * **Fault Type:** `ConnectionDrop`
   * **URL Pattern:** `*.bundle`
   * **Trigger Byte Offset:** `131072` (Drops connection after 128 KB transferred).
5. Check **"Enable Profile"**.

---

## Step 4: Run the Rehearsal in the Control Room

1. Open `Tools > PatchProof > Rehearsal Room`.
2. Configure your test run:
   * **Baseline Release:** `v1.0.0`
   * **Candidate Release:** `v2.0.0`
   * **Content Contract:** `CoreGameContract`
   * **Client Mode:** `Instrumented Binary`
   * **Fault Profile:** `MobileSubwayDropout`
3. Click **"Run Full Rehearsal Matrix"**.

### What Happens Behind the Scenes:
1. **Origin Startup:** PatchProof launches its internal HTTP server on `http://127.0.0.1:8899/`.
2. **Process Spawn:** It launches `Game_v1.0.0.exe` with command-line arguments:
   ```
   Game_v1.0.0.exe --patchproof-port 8898 --patchproof-token run_89a1bc4f
   ```
3. **Probe Handshake:** The Editor connects to the runtime probe on port `8898`.
4. **Cache Setup:** The probe initializes an isolated cache at `PatchProof_Runs/run_89a1bc4f/Cache/`.
5. **Addressables Redirection:** The probe overrides Addressables internal catalog URLs to point to the local staging origin.
6. **Update Execution:** The client invokes `Addressables.CheckForCatalogUpdates()` and `Addressables.UpdateCatalogs()`.
7. **Fault Trigger:** As the client downloads `Hero_Knight.bundle`, the origin abruptly closes the socket after 128 KB.
8. **Assertion:** The client probe records whether the Addressables runtime handled the error gracefully or entered an unhandled exception state.

---

## Step 5: Review Evidence & Results

Once the run completes, inspect the **Coverage Matrix**:
* **Tile 1 (Fresh Cache / Clean Network):** Green `PASSED`.
* **Tile 2 (Warm Cache / Clean Network):** Green `PASSED`.
* **Tile 3 (Fresh Cache / Mobile Dropout):** Red `FAILED` or Amber `UNVERIFIED` depending on whether your game's retry logic successfully recovered.

Click any tile to open the **Request Trace Inspector**:
* View exact HTTP request headers, response codes, and timeline watermarks.
* Verify whether bundles were served from cache or streamed over the wire.
* Confirm that no production URLs or access tokens leaked into the logs.

> ### Pro Tip: Exporting Repro-Packs
> If a scenario fails, click **"Export Repro-Pack"** on the tile. PatchProof bundles the exact catalogs, fault configurations, and trace timelines into a standalone zip file that any engineer on your team can unpack and debug.
