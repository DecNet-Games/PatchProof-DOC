---
layout: default
title: ReleaseManifest API
parent: API Reference
nav_order: 3
permalink: /docs/api/release-manifest/
---

# `ReleaseManifest` Data Contract
{: .no_toc }

Namespace: `Decnet.PatchProof.Core`  
Assembly: `Decnet.PatchProof.Core.dll`
{: .lead }

The `ReleaseManifest` class represents an immutable, cryptographically validated snapshot of a software and content release.

---

## Class Definition

```csharp
[Serializable]
public class ReleaseManifest
{
    public string releaseId;           // Primary unique key (e.g., "v1.0.0")
    public string releaseName;         // Human-readable title
    public TargetPlatform platform;    // WindowsStandalone, Android, etc.
    public string unityVersion;        // Engine version string (e.g., "6000.5.4f1")
    public string addressablesVersion; // Addressables package version (e.g., "2.9.1")
    public string catalogRelativePath; // Path to catalog file relative to content directory
    public string contentDirectory;    // Absolute or project-relative folder containing bundles
    public string executablePath;      // Path to compiled standalone player binary
    public List<ArtifactEntry> artifacts = new List<ArtifactEntry>();
}
```

---

## `ArtifactEntry` Structure

Represents a single physical file within the release's content closure:

```csharp
[Serializable]
public class ArtifactEntry
{
    public string relativePath;   // e.g., "hero_knight_assets.bundle"
    public long fileSizeBytes;    // Exact byte size
    public string sha256Checksum; // Hexadecimal SHA-256 digest
    public bool isCatalog;        // True if this is the Addressables catalog
    public bool isHashFile;       // True if this is the primary .hash metadata
}
```

---

## Example: Programmatic Manifest Generation

```csharp
using Decnet.PatchProof.Core;
using Decnet.PatchProof.Core.Validation;

var manifest = new ReleaseManifest
{
    releaseId = "v2.1.0-hotfix",
    releaseName = "Patch 2.1.0 Hotfix",
    platform = TargetPlatform.WindowsStandalone,
    unityVersion = Application.unityVersion,
    addressablesVersion = "2.9.1",
    catalogRelativePath = "catalog_v2.1.0.json",
    contentDirectory = "ServerData/StandaloneWindows64",
    executablePath = "Builds/Game_v2.1.0.exe"
};

// Compute SHA-256 hashes for all files in the content directory
ArtifactValidator.PopulateArtifactEntries(manifest);

// Save manifest to disk
string json = JsonUtility.ToJson(manifest, true);
File.WriteAllText("Assets/PatchProof_Registry/Manifests/v2.1.0.json", json);
```
