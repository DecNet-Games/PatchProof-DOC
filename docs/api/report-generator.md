---
layout: default
title: ReportGenerator API
parent: API Reference
nav_order: 6
permalink: /docs/api/report-generator/
---

# `ReportGenerator` Class
{: .no_toc }

Namespace: `Decnet.PatchProof.Core.Reporting`  
Assembly: `Decnet.PatchProof.Core.dll`
{: .lead }

The `ReportGenerator` class compiles completed `CoverageMatrix` data into standalone HTML release reports and structured JSON files.

---

## Static Methods

### `GenerateHtmlReport`
Generates a complete, self-contained HTML release dossier.

```csharp
public static string GenerateHtmlReport(
    CoverageMatrix matrix,
    ReleaseManifest baseline,
    ReleaseManifest candidate,
    ContentContract contract
);
```

#### Returns
* `string`: Raw HTML string ready to be written to disk.

---

### `GenerateJsonReport`
Generates a machine-readable JSON summary for CI integration.

```csharp
public static string GenerateJsonReport(
    CoverageMatrix matrix,
    ReleaseManifest baseline,
    ReleaseManifest candidate,
    ContentContract contract
);
```

#### Returns
* `string`: Serialized JSON string conforming to the PatchProof CI schema.

---

## Example: Exporting Reports from Code

```csharp
using Decnet.PatchProof.Core.Reporting;

// Generate reports after matrix execution
string html = ReportGenerator.GenerateHtmlReport(matrix, baseline, candidate, contract);
string json = ReportGenerator.GenerateJsonReport(matrix, baseline, candidate, contract);

File.WriteAllText("Reports/Release_v2.0.0.html", html);
File.WriteAllText("Reports/Release_v2.0.0.json", json);
```
