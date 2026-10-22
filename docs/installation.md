---
layout: default
title: Installation
nav_order: 3
permalink: /docs/installation/
has_children: false
---

# Installing PatchProof
{: .no_toc }

Comprehensive setup guide for purchasing, importing, and verifying PatchProof in your Unity project.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Commercial Licensing & Acquisition

> ### Commercial Distribution Notice
> PatchProof is a commercial developer tool available exclusively through the **[Unity Asset Store](https://assetstore.unity.com/packages/slug/421018)** (Short link: **[u3d.as/4g0P](https://u3d.as/4g0P)**). It is not open-source software, and full source code packages are distributed only to verified purchasers.

### Purchasing from the Asset Store
1. Visit the **Unity Asset Store** product listing:
   * **Full URL:** [https://assetstore.unity.com/packages/slug/421018](https://assetstore.unity.com/packages/slug/421018)
   * **Short Link:** [https://u3d.as/4g0P](https://u3d.as/4g0P)
2. Complete the purchase using your Unity Organization account.
3. Open your project in **Unity 6000.5.4f1 LTS** (or supported LTS version).
4. Navigate to `Window > Package Manager`.
5. In the Package Manager dropdown, select **Packages: My Assets**.
6. Locate **PatchProof**, click **Download**, and then click **Import**.

---

## System & Engine Requirements

PatchProof interacts directly with Unity's low-level compilation pipelines and process orchestration APIs. Verify your environment against these requirements:

| Component | Minimum Requirement | Recommended Specification |
|:---|:---|:---|
| **Unity Engine** | Unity 6000.0.0f1+ LTS | **Unity 6000.5.4f1 LTS** |
| **Addressables Package** | `com.unity.addressables` 2.0.0+ | **Addressables 2.9.1** (Bundled in Unity 6 LTS) |
| **Operating System** | Windows 10/11 (64-bit) | Windows 11 Pro (64-bit) |
| **Scripting Backend** | Mono or IL2CPP | Mono for fast test builds; IL2CPP for final twin validation |
| **Network Interfaces** | Local loopback (`127.0.0.1`) enabled | No firewall blocks on ephemeral ports (8000–9500) |

---

## Package Structure & Assembly Layout

When imported into your project, PatchProof organizes its codebase into clear, modular assemblies under `Assets/Decnet/PatchProof/`:

```
Assets/Decnet/PatchProof/
├── Adapters/
│   ├── CcdEndpointAdapter.cs           # Unity Cloud Content Delivery adapter & credential redaction
│   └── HistoricalClientDescriptor.cs   # Metadata definition for older client releases
├── Core/
│   ├── Comparison/                     # Catalog & code schema compatibility checkers
│   ├── Models/                         # Data models (ReleaseManifest, ContentContract, CoverageMatrix)
│   ├── Reporting/                      # HTML & JSON promotion report generators
│   ├── Rollback/                       # Dependency closure & rollback safety planners
│   └── Validation/                     # SHA-256 artifact hashing & path traversal guards
├── Editor/
│   ├── CI/                             # PatchProofHeadlessRunner batchmode entry points
│   ├── Orchestration/                  # ProcessManager, CacheManager, and RehearsalRunner
│   ├── Setup/                          # Quick setup wizard & sample laboratory generator
│   └── UI/                             # 7 dockable Editor windows (Rehearsal Room, Registry, etc.)
├── Origins/
│   └── LocalStagingOrigin.cs           # Embedded HTTP server with byte-level fault injection
├── RuntimeProbe/
│   ├── PatchProofProbe.cs              # MonoBehaviour probe injected into test binaries
│   └── ProbeProtocol.cs                # JSON-RPC command protocol over HTTP
├── Samples/
│   ├── Content/                        # Sample Addressables assets (Prefabs, Materials, Textures)
│   └── Scripts/                        # Runtime UI and sample room scripts
└── Tests/
    ├── Editor/                         # 29 independently verified EditMode tests
    └── Runtime/                        # 2 PlayMode verification tests
```

---

## Step-by-Step Project Setup

### Step 1: Verify Addressables Installation
Ensure the official Addressables package is active in your project:
1. Open `Window > Package Manager`.
2. Verify that **Addressables** (version `2.9.1` or compatible) is listed under **In Project**.
3. If not installed, add it via **Add package by name:** `com.unity.addressables`.

### Step 2: Run Guided Setup Wizard
PatchProof includes an automated onboarding wizard to configure project settings:
1. In the top menu, click:
   ```
   Tools > PatchProof > Guided Setup Wizard
   ```
2. The wizard checks:
   * Addressables settings asset existence.
   * Remote catalog build and load path configurations.
   * Local origin port availability (`8899` default).
   * Standalone build settings.
3. Click **"Apply Recommended Settings"** to initialize default folders and profiles.

### Step 3: Run the Verification Suite
To confirm that PatchProof has imported cleanly and compiled without conflicts in your project:
1. Open `Window > General > Test Runner`.
2. Under the **EditMode** tab, filter by assembly: `Decnet.PatchProof.Tests.Editor`.
3. Click **Run All**.
4. Confirm all **29 EditMode tests** pass.

> ### Pro Tip: Clean Git Ignore Rules
> PatchProof generates transient rehearsal runs, HTTP traces, and cached bundles during test executions. Add the following line to your `.gitignore`:
> ```
> # PatchProof transient rehearsal sandboxes
> PatchProof_Runs/
> ```
