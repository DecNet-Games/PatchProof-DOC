---
layout: default
title: Rehearsing a Live Update
parent: Workflows
nav_order: 2
permalink: /docs/workflows/rehearsing-an-update/
---

# Rehearsing a Live Update
{: .no_toc }

Standard operating procedure for verifying a candidate patch against your currently live production release.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## When to Use This Workflow

Execute this workflow whenever your studio is preparing to deploy an Addressables patch to live players:
* A weekly content drop (new cosmetics, skins, banners).
* A mid-season balance patch modifying gameplay prefabs.
* An urgent hotfix repairing a broken asset.

---

## The Pre-Rehearsal Checklist

- [ ] The **Baseline Release** in your Release Registry reflects the **exact build currently running on player devices**.
- [ ] Candidate Addressables content was built using **"Update a Previous Build"** targeting the baseline's `addressables_content_state.bin`.
- [ ] No git branch merges occurred that altered C# script GUIDs without updating the Content Contract.
- [ ] The local workstation has at least 5 GB of free disk space for isolated test sandboxes.

---

## Phase 1: Static Pre-Flight Audit

Before running binaries:
1. Open `Tools > PatchProof > Release Registry`.
2. Select your live baseline and candidate releases.
3. Click **"Compare Catalogs & Code Schemas"**.
4. Check the **Contract Difference Pane**:
   * Are there any unintentionally deleted keys?
   * Did any asset change its primary C# type?
   * Which bundles are unchanged (cache reusable) vs modified (wire transfer)?

If breaking changes appear, stop here. Fix the catalog or update your code before continuing.

---

## Phase 2: Dynamic Execution in the Rehearsal Room

1. Open `Tools > PatchProof > Rehearsal Room`.
2. Select the live baseline and candidate release pairing.
3. Set **Client Mode** to `Instrumented Binary`.
4. Click **"Run Full Rehearsal Matrix"**.

### Critical Matrix Scenarios to Inspect:

#### 1. Tile 2: Warm Cache / Clean Network
* **Verification Objective:** Confirms that existing players who already have yesterday's game files only download the modified delta bundles.
* **Inspect:** Open request traces and verify that `Transferred Wire Bytes` is significantly smaller than the total content size.

#### 2. Tile 4: Fresh Cache / Connection Drop
* **Verification Objective:** Confirms that if a player's connection drops during the download, re-launching the app recovers cleanly without a redownload loop.

#### 3. Tile 6: Missing Bundle (Incomplete CDN Simulation)
* **Verification Objective:** Confirms that your game displays an actionable error message rather than a black screen if the CDN has replication lag.

---

## Phase 3: Promotion Decision

Once all 7 tiles finish:
1. Review the **Truth Score** in the header. It must read **100% Verified**.
2. Click **"Export Release Report"** and save the HTML report to your team's release repository.
3. Send the HTML report to your Lead Producer or Release Manager.
4. With full verification established, push the candidate bundles to your production CDN with complete confidence!
