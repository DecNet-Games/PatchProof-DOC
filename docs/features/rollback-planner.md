---
layout: default
title: Rollback Planner
parent: Features Overview
nav_order: 10
permalink: /docs/features/rollback-planner/
---

# Rollback Planner
{: .no_toc }

Dependency closure computation, missing-history trap prevention, and safe emergency rollbacks.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Rollback Illusion

When a new content update causes unexpected issues in production, the instinctive reaction of a live operations team is to **roll back the catalog**:
* They revert the CDN symlink or remote bucket catalog file back to `catalog_v1.0.0.json`.
* They assume players will simply reload the previous stable version.

In reality, **naive rollbacks frequently make outages worse**:
1. **Pruned Bundles:** To save storage space on the CDN, an automated build script may have pruned baseline bundles that the old catalog still depends on.
2. **Asymmetric Caching:** Players who already downloaded the bad v2.0.0 update now have modified bundles in their cache. When reverted to v1.0.0, the old catalog expects v1.0.0 hashes, but the filesystem contains v2.0.0 files, triggering unhandled CRC checksum mismatch crashes.
3. **Database Schema Drift:** If the candidate update migrated local save files or player profiles, the old client code may crash when reading the newer save data.

PatchProof's **Rollback Planner** ([`RollbackPlanner.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Rollback/RollbackPlanner.cs)) simulates and validates rollback safety before deployment.

---

## Dependency Closure Computation

Before promoting any release, the Rollback Planner computes the **Complete Dependency Closure** between both releases:

```
+-----------------------------------------------------------------------------+
|                          Rollback Closure Analysis                          |
+-----------------------------------------------------------------------------+
|   Candidate Release (v2.0.0) -> Reverted to -> Baseline Release (v1.0.0)    |
|                                                                             |
|   Required Baseline Bundles:      42 files                                  |
|   Available on Staging Origin:    42 files (100% Present)                   |
|   Missing Historical Artifacts:   0 files                                   |
|   Shared Immutable Bundles:       35 files                                  |
|   Replaced Mutated Bundles:        7 files                                  |
|                                                                             |
|   ROLLBACK CLOSURE STATUS: VERIFIED SAFE                                    |
+-----------------------------------------------------------------------------+
```

### Missing History Failure
If any bundle required by the baseline catalog is missing from the staging origin, PatchProof fails the rollback audit:
```
[RollbackPlanner] ERROR: Rollback closure incomplete!
Catalog 'catalog_v1.0.0.json' requires bundle 'shared_shaders_v1.bundle', 
but this file does not exist in the active content directory.
A live rollback to v1.0.0 would result in HTTP 404 crashes for all players!
```

---

## Rehearsing a Live Rollback

PatchProof includes a dedicated scenario tile specifically for rollback rehearsal:
* **Scenario Name:** `Rollback_BaselineRehearsal`
* **Test Sequence:**
  1. An instrumented player starts with a **Warm Cache** populated with candidate v2.0.0 assets.
  2. The local staging origin switches its primary catalog endpoint back to `v1.0.0`.
  3. The runtime probe calls `Addressables.CheckForCatalogUpdates()` and receives the v1.0.0 catalog metadata.
  4. The probe calls `Addressables.UpdateCatalogs()`.
  5. The client attempts to load the baseline asset suite.
  6. The probe asserts that Addressables successfully overwrote the newer cached bundles with baseline versions and that all baseline assets instantiate without errors.

---

## Safe Rollback Checklist

PatchProof automatically embeds a **Rollback Safety Checklist** in every exported release report:

- [ ] All baseline asset bundles are preserved in the CDN origin bucket (no aggressive TTL deletion).
- [ ] Catalog hash file updates propagate to edge CDN nodes within 60 seconds.
- [ ] Local client save game formats maintain backward compatibility with previous client code.
- [ ] Rollback scenario tile (`Rollback_BaselineRehearsal`) completed with status `PASSED` in the Rehearsal Room.

> ### Pro Tip: Multi-Version Rollbacks
> Need to support rolling back across multiple historical versions (e.g., from v1.3.0 back to v1.1.0)? Register all intermediate releases in the Release Registry and run the Rollback Planner across the multi-version chain.
