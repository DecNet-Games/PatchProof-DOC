---
layout: default
title: Limitations & Boundaries
nav_order: 10
permalink: /docs/limitations/
has_children: false
---

# Limitations & Verification Boundaries
{: .no_toc }

A complete, transparent disclosure of verified environments, platform constraints, and operational boundaries.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Engineering Honesty Policy

PatchProof is built on a strict policy of developer honesty. We distinguish plainly between what has been **implemented**, what has been **inspected**, and what has been **executed and proven**.

We will never claim compatibility that has not been verified on real hardware or claim guarantees that violate the laws of networking.

---

## 1. Engine & Package Version Fence

PatchProof was developed and verified against an exact engine and package combination:

* **Verified Unity Engine:** **Unity 6000.5.4f1 LTS** (Windows x86_64).
* **Verified Addressables Version:** **Addressables 2.9.1** (Bundled in Unity 6 LTS).

> ### Important Boundary
> Do not assume compatibility with legacy Addressables versions (e.g., `1.19.x` on Unity 2020 LTS) or future alpha/beta Addressables releases. APIs for catalog serialization and `Addressables.UpdateCatalogs` have evolved significantly between major releases. PatchProof is strictly verified for Unity 6 LTS with Addressables 2.9.1.

---

## 2. Platform Status Disclosures

### Windows Standalone (x86_64) — VERIFIED
* **Status:** **Fully Verified**.
* **Capabilities:** Process spawning, run-token isolation, loopback origin streaming, probe JSON-RPC handshake, cache sandboxing, in-memory contract evaluation, and clean process termination are 100% verified and tested on Windows Standalone.

### Android Platform — CONFIGURATION ONLY, NOT VERIFIED
* **Status:** **Configuration Only — NOT VERIFIED**.
* **What Ships:** Android manifest schemas, target platform enum entries, ADB pairing port configurations, and device descriptor structures ship in source.
* **What is NOT Verified:** **Physical Android device behavior has NOT been verified.** PatchProof does not claim automated device deployment, USB socket forwarding, or Android-specific filesystem sandboxing. The tool must never be presented as a verified Android runtime rehearsal solution.

### macOS & Linux Standalone — UNVERIFIED
* **Status:** **Unverified**.
* `LocalStagingOrigin` uses standard .NET `HttpListener` and `Process` APIs which are portable, but automated execution has not been verified on macOS or Linux host workstations.

---

## 3. Automated Test Suite Boundaries

PatchProof ships with **31 automated tests**:

```
+-----------------------------------------------------------------------------+
| PatchProof Automated Test Suite Breakdown                                   |
+-----------------------------------------------------------------------------+
|   EditMode Tests (29 tests):  Independently Verified Passing (29/29)        |
|   PlayMode Tests (2 tests):   Per Project Evidence Only (Not Reproducible)  |
|   TOTAL SHIPPED TESTS:        31 Tests Total                                |
+-----------------------------------------------------------------------------+
```

### EditMode Test Suite (29 Tests)
* **Status:** **29/29 Passed**.
* Independently re-run and confirmed in batchmode on Unity 6000.5.4f1 LTS.
* Covers schema hashing, historical client modes, catalog contract evaluation, cache matrix sandboxing, stale catalog detection, rollback dependency closures, process isolation, timeout cancellation, staging endpoint privacy, Repro-Pack generation, and CI promotion gating.

### PlayMode Test Suite (2 Tests)
* **Status:** **Per Project Evidence Only — Not Independently Reproduced**.
* Two tests ship in `PP_T05_InterruptedFetchTests.cs` (`Origin_CleanRequest_ServesExactFileBytesAndRecordsTrace` and `Origin_Injected404Fault_ReturnsHttp404`).
* While these passed in the project's recorded local runs, headless PlayMode batchmode test execution hangs on this environment's display adapter setup. They are **not independently reproduced** in CI, and documentation must state this plainly.

---

## 4. Spec Acceptance Coverage Truth

* **Verified Spec Areas:** **11 of 12 spec test areas are fully verified**.
* **The 12th Area (PP-T12, Integrated Buyer Journey):** **Partially verified at component level only**. The individual building blocks (headless CI runner, promotion gating, HTML/JSON export, quick setup wizard) pass unit testing, but the entire end-to-end buyer first-use journey has not been executed as an uninterrupted continuous batch run.
* **Never "24/24":** Blanket claims like "24/24 requirements verified" or "all features proven complete" are strictly prohibited across all documentation.

---

## 5. Exclusions & Operational Non-Goals

1. **PatchProof is NOT a CDN:** It does not host, distribute, or sync files to cloud providers (AWS, Azure, Cloudflare, Fastly). It is an in-Editor local staging server that simulates cloud delivery on localhost.
2. **Never Touches Production Endpoints:** PatchProof never connects to, mutates, or modifies live player endpoints or active production buckets.
3. **No Live-Rollback Guarantee:** PatchProof plans and verifies rollbacks on local staging data. It cannot guarantee that your external cloud bucket permissions or third-party edge cache purges will succeed in production.
4. **No Binary Code Patching:** PatchProof tests content updates against existing client binaries. It cannot patch compiled C# assembly bytecode in an already-installed mobile APK or PC executable.
