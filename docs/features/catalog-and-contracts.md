---
layout: default
title: Catalogs & Contracts
parent: Features Overview
nav_order: 9
permalink: /docs/features/catalog-and-contracts/
---

# Catalogs & Content Contracts
{: .no_toc }

Static catalog comparison, breaking change detection, and C# type schema compatibility checks.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Static Pre-Flight Analysis

Before launching a standalone player executable to run dynamic network rehearsals, PatchProof can perform deep **static pre-flight analysis** between baseline and candidate releases.

Implemented in [`CatalogComparer.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Comparison/CatalogComparer.cs) and [`CodeCompatibilityChecker.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Comparison/CodeCompatibilityChecker.cs), this tool evaluates catalogs directly in the Editor in milliseconds.

---

## What Static Comparison Detects

```
+-----------------------------------------------------------------------------+
|                            Catalog Comparison Engine                        |
+-----------------------------------------------------------------------------+
|   1. Dropped Keys           | Keys present in baseline but missing in new.  |
|   2. Type Modifications     | Asset type changed (e.g., GameObject->Audio)  |
|   3. Bundle Hash Drift      | Inadvertent bundle re-serialization.          |
|   4. Missing Assembly Types | C# MonoBehaviour class renamed or deleted.    |
+-----------------------------------------------------------------------------+
```

### 1. Dropped Primary Keys
If an Addressables key was present in the baseline release (v1.0.0) but does not appear in the candidate catalog (v2.0.0), PatchProof flags it:
* If the key is listed in the Content Contract's `permittedRemovals`, it is marked as an **Expected Deprecation**.
* If the key is not in `permittedRemovals`, PatchProof reports a **Breaking Contract Violation**.

### 2. Type Mutation Violations
If key `"Prefabs/Weapon"` mapped to `UnityEngine.GameObject` in v1.0.0, but now points to `MyGame.Data.WeaponConfig` in v2.0.0, existing code expecting a GameObject will fail at runtime. `CatalogComparer` identifies this type mutation before runtime.

### 3. Inadvertent Bundle Hash Drift
When building an Addressables content update, ideally only modified asset groups should produce new bundles. If an asset was merely re-saved without changes, but its internal hash drifted due to shader compilation variations, `CatalogComparer` displays:
```
Group: Environment_Forest
Status: Hash Drift Detected (Zero asset changes, but bundle CRC changed)
Impact: Forces 45 MB redownload for existing players
```

---

## Code & Schema Compatibility Checks

One of the most catastrophic failures in Unity Addressables occurs when an asset bundle references a C# class whose serialization layout changed between engine builds:

```
[MissingComponentException: The script serialized in bundle 'enemies_boss.bundle' 
with GUID 'a891bf...' cannot be found in the current assembly.]
```

PatchProof's `CodeCompatibilityChecker` scans the type tables embedded inside candidate asset bundles:
1. It extracts every referenced `MonoScript` type name and assembly definition name.
2. It cross-references them against the loaded assemblies in the historical client binary descriptor.
3. If an assembly or class was renamed, moved to a different namespace, or removed, PatchProof halts the release process with an exact list of missing types and affected bundle files.

---

## Running Catalog Comparison in the Editor

1. Open `Tools > PatchProof > Release Registry`.
2. Select your **Baseline Release** in the left panel.
3. Select your **Candidate Release** in the right panel.
4. Click **"Compare Catalogs & Code Schemas"**.
5. The comparison inspector displays:
   * **Total Keys:** Baseline vs Candidate count.
   * **Unchanged Bundles:** Reusable from cache.
   * **Updated Bundles:** Transferred over wire.
   * **Contract Status:** 100% Valid or Breaking Violations Listed.

> ### Pro Tip: CI Pre-Flight Gate
> In automated pipelines, run `CatalogComparer` as an immediate fast-fail pre-flight step. If breaking type changes are discovered, the pipeline aborts in 2 seconds without waiting to compile or launch standalone player executables!
