---
layout: default
title: Release Registry
parent: Features Overview
nav_order: 2
permalink: /docs/features/release-registry/
---

# Release Registry & Immutability
{: .no_toc }

Managing historical releases, candidate artifacts, and cryptographic SHA-256 validation.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Immutability is Mandatory

In content delivery engineering, accidental in-place modifications are a common source of unreproducible bugs:
* An engineer rebuilds an Addressables group without changing the release version, overwriting a bundle on disk while the catalog hash remains cached.
* A file compression utility modifies archive timestamps, invalidating client cache timestamps.
* A developer tests against bundles that do not match the compiled candidate executable.

PatchProof's **Release Registry** (`Tools > PatchProof > Release Registry`) solves this by treating every release as an **immutable, cryptographically signed snapshot**.

---

## The Release Manifest

Implemented in [`ReleaseManifest.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Models/ReleaseManifest.cs), a Release Manifest encapsulates all physical and metadata properties of a version:

```csharp
[Serializable]
public class ReleaseManifest
{
    public string releaseId;           // e.g., "v1.0.4"
    public string releaseName;         // Friendly display title
    public TargetPlatform platform;    // TargetPlatform.WindowsStandalone
    public string unityVersion;        // e.g., "6000.5.4f1"
    public string addressablesVersion; // e.g., "2.9.1"
    public string catalogRelativePath; // Path to catalog_v1.0.4.json
    public string contentDirectory;    // Folder containing .bundle files
    public string executablePath;      // Path to Standalone Player .exe
    public List<ArtifactEntry> artifacts = new List<ArtifactEntry>();
}
```

---

## Artifact Integrity Hashing

When you save a release manifest in the registry, PatchProof's [`ArtifactValidator`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Core/Validation/ArtifactValidator.cs) scans the entire content directory:

1. **SHA-256 Checksumming:** Reads every bundle, catalog, and hash file in 64 KB buffers, computing an unforgeable SHA-256 hash.
2. **Size Recording:** Stores exact file byte lengths.
3. **Immutability Locking:** Saves the artifact manifest into `Assets/PatchProof_Registry/Manifests/{releaseId}.json`.

```
File: hero_knight_assets_all.bundle
Size: 14,832,109 bytes
SHA-256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

### Automatic Divergence Detection
Before every rehearsal run, PatchProof re-hashes the files on disk:
* If a bundle's size or SHA-256 digest has changed since registration, the run is aborted with `FailureCategory.ArtifactIntegrityMismatch`.
* This prevents staging rehearsals against dirty, uncommitted build artifacts.

---

## Registering a Release in the Editor

1. Open `Tools > PatchProof > Release Registry`.
2. Click **"Register Release"**.
3. Choose the source:
   * **From Active Build Output:** Automatically detects the current Addressables build output in `ServerData/`.
   * **From Archived Directory:** Browse to any previously built release folder on your drive or network share.
4. Assign a unique **Release ID** (e.g., `v1.2.0`).
5. Specify the path to the companion Standalone `.exe` player.
6. Click **"Compute Hashes & Lock Manifest"**.

> ### Pro Tip: Archiving Historical Binaries
> Store historical release binaries in a shared network folder (e.g., `P:/Builds/HistoricalClients/`). Your entire engineering team can point their Release Registries to this folder to rehearse against identical historical baselines.
