---
layout: default
title: Probe API & Protocol
parent: API Reference
nav_order: 2
permalink: /docs/api/probe-api/
---

# `PatchProofProbe` & Protocol API
{: .no_toc }

Namespace: `Decnet.PatchProof.RuntimeProbe`  
Assembly: `Decnet.PatchProof.RuntimeProbe.dll`
{: .lead }

The `PatchProofProbe` component runs inside standalone player binaries, providing a loopback HTTP JSON-RPC endpoint for Editor commands.

---

## Component Definition

```csharp
[DefaultExecutionOrder(-1000)]
public class PatchProofProbe : MonoBehaviour
{
    public static PatchProofProbe Instance { get; private set; }
    public int ProbePort { get; }
    public bool IsServerRunning { get; }

    public void EnqueueMainThread(Action action);
    public Task<ProbeResponse> ExecuteCommandAsync(ProbeRequest request);
    public void StopProbeServer();
}
```

---

## The Command Protocol (`ProbeRequest`)

```csharp
[Serializable]
public class ProbeRequest
{
    public ProbeCommandType command;
    public string runToken;
    public string endpointOverrideUrl;
    public CacheState desiredCacheState;
    public string contractJson;
}

public enum ProbeCommandType
{
    Handshake,
    PrepareOwnedCache,
    InitializeAddressables,
    CheckForUpdate,
    UpdateCatalogs,
    AssertContract,
    CleanShutdown
}
```

---

## The Response Protocol (`ProbeResponse`)

```csharp
[Serializable]
public class ProbeResponse
{
    public bool success;
    public string message;
    public string engineVersion;
    public string addressablesVersion;
    public string platform;
    public List<ClientAssertion> assertions = new List<ClientAssertion>();
    public List<ProbeEvent> events = new List<ProbeEvent>();
}

[Serializable]
public class ClientAssertion
{
    public string key;
    public string expectedType;
    public string actualType;
    public bool passed;
    public string failureReason;
}
```

---

## Example: Invoking the Probe over HTTP

You can interact with the probe directly using any HTTP client (e.g., `curl` or PowerShell) when debugging standalone binaries:

```bash
curl -X POST http://127.0.0.1:8898/ \
  -H "Content-Type: application/json" \
  -d '{"command": 0, "runToken": "manual_debug_session"}'
```

Response:
```json
{
  "success": true,
  "message": "Probe handshake successful.",
  "engineVersion": "6000.5.4f1",
  "addressablesVersion": "2.9.1",
  "platform": "WindowsPlayer"
}
```
