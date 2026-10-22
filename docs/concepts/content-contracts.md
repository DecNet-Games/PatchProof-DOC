---
layout: default
title: Semantic Content Contracts
parent: Core Concepts
nav_order: 6
permalink: /docs/concepts/content-contracts/
---

# Semantic Content Contracts
{: .no_toc }

Beyond string keys: asserting type safety, component presence, and schema compatibility across releases.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The String-Key Illusion

In Unity Addressables, asset keys are arbitrary strings (e.g., `"Prefabs/Player"`, `"Audio/Level1Music"`, or GUIDs). 

This design creates a major operational hazard: **The catalog can successfully find a key, download the bundle, and instantiate an object — yet completely break the game at runtime.**

Consider these common release disasters:
* An artist re-exports `"Prefabs/Weapon_Sword"` as a `Texture2D` icon instead of a `GameObject` prefab. Addressables loads it without error, but your inventory script crashes with an `InvalidCastException`.
* A refactoring engineer renames a C# script from `CharacterController` to `PlayerMovementController`. The prefab instantiates, but the attached script becomes `Missing Component (MonoBehaviour)`.
* A sound designer deletes a key `"Audio/VictoryFanfare"` that a victory screen script requires, resulting in an unhandled `KeyNotFoundException` during a match climax.

PatchProof prevents these issues through **Semantic Content Contracts**.

---

## Contract Architecture

A [`ContentContract`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/ContentContract.cs) formalizes what assets must exist and what invariants they must satisfy:

```json
{
  "name": "CoreGame_ReleaseContract",
  "version": "2.0.0",
  "requiredEntries": [
    {
      "addressableKey": "Prefabs/MainMenu",
      "expectedType": "UnityEngine.GameObject",
      "requiredComponents": [
        "MyGame.UI.MainMenuController",
        "UnityEngine.Canvas"
      ],
      "allowNull": false
    },
    {
      "addressableKey": "Audio/ThemeMusic",
      "expectedType": "UnityEngine.AudioClip",
      "allowNull": false
    }
  ],
  "permittedRemovals": [
    "Prefabs/LegacyMenu_v1"
  ]
}
```

---

## Contract Evaluation Engine

When the runtime probe receives the `AssertContract` command, it executes a rigorous verification sequence inside the standalone player:

```
+-----------------------------------------------------------------------------+
|                          Probe Assertion Sequence                           |
+-----------------------------------------------------------------------------+
|  1. Key Resolution: Does Addressables know about the declared key?         |
|  2. Async Load: Invoke Addressables.LoadAssetAsync<T>(key)                 |
|  3. Type Assertion: Is loadedObject.GetType() == expectedType?              |
|  4. Component Inspection (Prefabs):                                         |
|     - Does prefab.GetComponent(compType) != null?                           |
|     - Are there any MissingScript components on any child Transform?        |
|  5. Release Memory: Addressables.Release(handle)                            |
+-----------------------------------------------------------------------------+
```

### Static Pre-Flight Analysis
Even before launching a client binary, PatchProof's [`CatalogComparer`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Comparison/CatalogComparer.cs) performs static analysis between baseline and candidate catalogs:
* Detects silently dropped keys.
* Flags internal bundle dependency changes.
* Identifies C# type serialization hash mismatches across engine builds.

> ### Pro Tip: Permitted Removals
> When deprecating old content, always list removed keys in `permittedRemovals`. PatchProof will verify that no remaining candidate assets have dangling internal dependencies on those deleted keys.
