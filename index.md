---
layout: default
title: Home
nav_order: 1
description: "PatchProof — Addressables Release Rehearsal Lab for Unity."
permalink: /
---

# PatchProof
{: .fs-9 }

### Addressables Release Rehearsal Lab for Unity
{: .fs-6 .text-gamma }

Rehearse your Addressables update the way your oldest installed client will live it — fresh cache, warm cache, broken network, rollback — before your players do.
{: .lead }

[Get Started Now]({{ site.baseurl }}/docs/getting-started/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on Asset Store (u3d.as/4g0P)](https://u3d.as/4g0P){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## The Reality of Addressables Updates

### The Production Dilemma
You tested your Addressables build in the Editor with "Use Existing Build". It loaded fine. Your CI build succeeded. You pushed your catalog and asset bundles to your CDN. 

Twenty minutes later, your live ops dashboard explodes:
* Existing players on version 1.0.4 crash on launch because their local cache references bundle hash metadata that the new catalog invalidated.
* A player on mobile subway connectivity downloads half of the new asset catalog, loses signal, re-launches the game, and enters an unrecoverable `InvalidKeyException` loop.
* Your operations team triggers a catalog rollback to the previous release, only to discover that the old catalog references bundles that your cleanup script pruned from the bucket.

> ### The Problem: Untested Client State Transitions
> Most update bugs do not happen in clean installs. They happen during **state transitions** — an existing client on an older engine build or schema running against a modified catalog over imperfect real-world networks.

---

## What PatchProof Does

PatchProof is an **in-Editor release control room and local staging laboratory** that executes update scenarios against live standalone runtime binaries before you upload a single byte to your production CDN.

```
+-----------------------------------------------------------------------------+
|                           PatchProof Control Room                           |
+-----------------------------------------------------------------------------+
|  Release Registry  |  Content Contracts  |  Staging Origin  |  Fault Engine |
+-----------------------------------------------------------------------------+
                                       |
                   Dispatches Scenarios over Local Loopback
                                       v
+-----------------------------------------------------------------------------+
|                     Owned Standalone Runtime Binary                         |
|                 (Running Historical Release e.g. v1.0.0)                    |
+-----------------------------------------------------------------------------+
|  Step 1: Handshake Probe API                                                |
|  Step 2: Prepare Isolated Cache (Fresh / WarmOld / MixedPartial)             |
|  Step 3: Redirect Addressables to Local Staging Origin (127.0.0.1:8899)     |
|  Step 4: Check & Download Candidate Release Catalogs & Bundles              |
|  Step 5: Load Content & Verify Semantic Type Contracts                      |
|  Step 6: Stream Request Telemetry & Assertions Back to Editor Matrix        |
+-----------------------------------------------------------------------------+
```

---

## Core Pillars

### 1. Zero CDN Dependency & Absolute Endpoint Safety
PatchProof is **NOT a CDN** and never requires cloud credentials or internet access to test updates. It runs an embedded, high-performance HTTP staging origin bound strictly to `127.0.0.1`. All URLs, tokens, and query strings are automatically redacted before logging.

### 2. Physical Cache Matrix Isolation
Rehearsals run across reproducible cache topologies:
* **Fresh Cache:** Simulates a clean install or cleared application storage.
* **Warm Old Cache:** Populates the client's storage with valid bundles from the baseline release before triggering the update.
* **Mixed / Partial Cache:** Simulates aborted prior downloads with dangling payload files.
Every scenario executes in an isolated sandbox folder keyed by an owned run token to prevent cross-run pollution.

### 3. Byte-Boundary Network Fault Injection
Real players encounter network drops. PatchProof's origin server injects faults at precise byte boundaries:
* **Timeouts & Latency:** Delay response headers or content chunks.
* **Connection Drops:** Abort TCP sockets mid-transfer after $N$ bytes.
* **HTTP Errors:** Return 404 Not Found, 403 Forbidden, or 500 Server Error for specific bundle patterns.
* **Corrupted Payloads & Truncation:** Flip bits or truncate downloads to verify checksum validation.

### 4. Semantic Content Contracts
Addressables keys are just strings, but asset types break across code changes. PatchProof evaluates **Content Contracts** that assert not just that a key loads, but that the instantiated object matches expected C# types, component schemas, and interface implementations.

### 5. Honest Promotion Gating (No Empty Green Matrices)
If a scenario tile was not executed with an active probe verifying runtime memory, PatchProof marks it **UNVERIFIED**. The headless CI runner returns exit code `3` if any unverified cells remain. PatchProof refuses to sugarcoat coverage.

---

## Technical Specifications & Verification Fence

PatchProof is engineered with strict adherence to verified facts:

| Parameter | Tested & Verified Specification |
|:---|:---|
| **Unity Version** | Developed and tested on **Unity 6000.5.4f1 LTS** |
| **Addressables Version** | Tested with **Addressables 2.9.1** (Unity 6 LTS bundled) |
| **Runtime Rehearsal Target** | **Windows Standalone (x86_64)** via instrumented binary & loopback probe |
| **Android Status** | **Configuration only — NOT VERIFIED** (schemas, fixtures, pairing configuration ship; physical device execution is not verified) |
| **PlayMode Rehearsal** | 2 PlayMode tests ship in source and passed recorded project runs (**per project evidence only — not independently reproduced**) |
| **Automated Tests** | **31 total tests** ship (29 EditMode + 2 PlayMode). The EditMode suite is **29/29 independently passing** |
| **Spec Acceptance Coverage** | **12/12 spec acceptance areas verified** (11 via EditMode suite, 1 via PlayMode project evidence) |
| **Store Links** | [Unity Asset Store Package](https://assetstore.unity.com/packages/slug/421018) • [Short Link: u3d.as/4g0P](https://u3d.as/4g0P) |

---

## Quick Navigation

<div class="cards">
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/getting-started/">Getting Started</a></h3>
      <p>Understand the core mental model: Register &rarr; Declare &rarr; Set Conditions &rarr; Execute &rarr; Inspect Evidence.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/installation/">Installation</a></h3>
      <p>How to acquire PatchProof from the Unity Asset Store and configure your project dependencies.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/">Feature Deep Dives</a></h3>
      <p>Explore all 13 core modules: Rehearsal Room, Fault Engine, Cache Manager, Rollback Planner, and CI Runner.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/workflows/first-rehearsal/">Step-by-Step Workflows</a></h3>
      <p>Practical guides for setting up candidate updates, simulating CDN outages, and wiring headless CI pipelines.</p>
    </div>
  </div>
</div>
