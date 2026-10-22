---
layout: default
title: ContentContract API
parent: API Reference
nav_order: 4
permalink: /docs/api/content-contract/
---

# `ContentContract` API
{: .no_toc }

Namespace: `Decnet.PatchProof.Core`  
Assembly: `Decnet.PatchProof.Core.dll`
{: .lead }

The `ContentContract` class defines programmatic rules governing required Addressables keys, expected C# types, and component invariants.

---

## Class Definition

```csharp
[Serializable]
public class ContentContract
{
    public string name = "New Content Contract";
    public string targetReleaseVersion = "2.0.0";
    public List<ContractEntry> requiredEntries = new List<ContractEntry>();
    public List<string> permittedRemovals = new List<string>();
}

[Serializable]
public class ContractEntry
{
    public string addressableKey;
    public string expectedType;                 // Full qualified System.Type name
    public List<string> requiredComponents;     // Required MonoBehaviour types on prefabs
    public bool allowNull = false;
    public string notes;
}
```

---

## Static Validation with `CatalogComparer`

You can programmatically validate a contract against an Addressables catalog snapshot without launching a game client:

```csharp
using Decnet.PatchProof.Core;
using Decnet.PatchProof.Core.Comparison;

ContentContract contract = LoadContractFromDisk("Assets/Contracts/Core.json");
CatalogSnapshot candidateCatalog = CatalogSnapshot.FromFile("ServerData/catalog.json");

ContractValidationResult result = CatalogComparer.ValidateContract(contract, candidateCatalog);

if (!result.IsValid)
{
    foreach (var missingKey in result.MissingKeys)
    {
        Debug.LogError($"Missing Contract Key: {missingKey}");
    }
}
```
