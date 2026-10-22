---
layout: default
title: Sample Laboratory
nav_order: 7
permalink: /docs/sample/
has_children: true
---

# Sample Update Laboratory
{: .no_toc }

A ready-to-run release sandbox included directly in PatchProof for evaluation, onboarding, and regression testing.
{: .lead }

To help teams understand Addressables release verification without touching their production assets, PatchProof includes a fully self-contained **Sample Laboratory**.

---

## What the Sample Lab Contains

The laboratory provides a complete, mini game environment designed specifically to exercise content updates:

```
Assets/Decnet/PatchProof/Samples/
├── Content/
│   ├── Prefabs/
│   │   ├── SampleRoom_v1.prefab       # Original baseline environment (v1.0.0)
│   │   ├── SampleRoom_v2.prefab       # Updated room with new props (v2.0.0 Valid)
│   │   └── SampleRoom_v2_Broken.prefab# Seeded with missing C# scripts (v2.0.0 Broken)
│   ├── Materials/                     # Textures and shader variants
│   └── Audio/                         # Ambient audio clips
└── Scripts/
    ├── LabDemoClientUI.cs             # Runtime HUD displaying active Addressables status
    └── SampleRoomEntity.cs            # Custom gameplay component evaluated by contracts
```

---

## One-Click Setup Wizard

Accessible via `Tools > PatchProof > Local Update Lab`:
1. **"Generate Sample Releases & Baseline Content":** Builds both `v1.0.0` and `v2.0.0` content catalogs and bundle packages into your project's local storage.
2. **"Build Instrumented Windows Binary":** Compiles the standalone player executable with the runtime probe attached.
3. **"Run Sample Verification Matrix":** Dispatches all 7 matrix scenarios against the sample binary, taking you from zero to full verification in 60 seconds.

Read the detailed guide in [Sample Laboratory Walkthrough]({{ site.baseurl }}/docs/sample/sample-laboratory/).
