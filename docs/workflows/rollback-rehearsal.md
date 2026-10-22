---
layout: default
title: Rollback Rehearsal Procedure
parent: Workflows
nav_order: 4
permalink: /docs/workflows/rollback-rehearsal/
---

# Rollback Rehearsal Procedure
{: .no_toc }

Standard procedure for rehearsing emergency catalog rollbacks and preventing missing-history outages.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Philosophy of "Rollback First"

At top-tier live-service game studios, **no release is permitted to ship unless the rollback plan has been actively tested**.

If an unforeseen game-breaking bug slips into a Friday release, the live operations team must be 100% certain that pointing the CDN back to the previous catalog will successfully restore the game without leaving active players stranded.

---

## The 4-Step Rollback Rehearsal

```
+-----------------------------------------------------------------------------+
|                          1. Audit Dependency Closure                        |
|   Ensure all baseline bundles still exist in the distribution directory.    |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                            2. Seed Client Cache                             |
|   Seed the client's local cache with the new candidate v2.0.0 bundles.      |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                           3. Revert Catalog Route                           |
|   Point the staging origin's primary catalog route back to v1.0.0.          |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                            4. Verify Client Memory                          |
|   Assert that Addressables cleanly rolls back and loads baseline assets.    |
+-----------------------------------------------------------------------------+
```

---

## Executing the Rollback Scenario in PatchProof

1. Open `Tools > PatchProof > Rehearsal Room`.
2. Select your baseline and candidate releases.
3. In the scenario matrix, locate **Tile 7: `Rollback_BaselineRehearsal`**.
4. Click **"Run Scenario"**.

### Automated Sequence:
1. PatchProof initializes the client player in an isolated sandbox and commands it to download and load the **Candidate (v2.0.0)** release. The cache is now "polluted" with new assets.
2. The staging origin dynamically modifies its route table:
   * `/content/catalog.json` &rarr; Points back to `baseline_catalog.json`.
3. The probe commands the client to check for updates.
4. The client receives the old catalog metadata.
5. Addressables identifies that the active bundles in cache do not match the old catalog's expected hashes.
6. Addressables fetches the baseline bundles from the origin and restores the previous content locator.
7. The probe asserts the **Baseline Content Contract**.
8. If the baseline room loads without errors, the tile turns green: **PASSED**.

---

## What to Do if Rollback Fails

If Tile 7 fails, common root causes include:
* **Missing Baseline Archives:** Someone deleted an older bundle file to clear space. Restore the file from your backup archive.
* **Persistent Save Game Corruption:** Your new candidate code saved data in a format that the old baseline code crashes on when reading. Ensure your game uses backward-compatible serialization protocols (e.g., protobuf or version-tagged JSON).
