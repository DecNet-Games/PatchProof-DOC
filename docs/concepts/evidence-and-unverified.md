---
layout: default
title: Evidence & Unverified States
parent: Core Concepts
nav_order: 3
permalink: /docs/concepts/evidence-and-unverified/
---

# Evidence & Unverified States
{: .no_toc }

Why PatchProof rejects empty green matrices and why unexecuted cells remain in the denominator.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The "False Green" Syndrome

In modern continuous delivery pipelines, green checkmarks can be deceptively dangerous:
* A test suite passes because an assertion was wrapped in an empty `try/catch`.
* An Addressables check passes because the test only validated that the catalog file exists on disk, never attempting to download or deserialize the actual bundle payload.
* An external process exits with code 0 even though Unity logged internal null reference exceptions during catalog activation.

PatchProof enforces an unbending engineering principle: **No release promotion without positive runtime evidence.**

---

## Scenario Status Lifecycle

Every cell in PatchProof's Coverage Matrix is represented by a `CoverageTile` object with four possible states:

```
                  +---------------------------+
                  |         PENDING           |
                  +---------------------------+
                                |
                  +-------------+-------------+
                  |                           |
                  v                           v
     [ Probe Connected? ]            [ External Mode / Fault Gap? ]
                  |                           |
         +--------+--------+                  v
         |                 |             +------------+
         v                 v             | UNVERIFIED |
     +--------+       +--------+         +------------+
     | PASSED |       | FAILED |
     +--------+       +--------+
```

### 1. `PASSED`
A tile is marked `PASSED` **only when all of the following conditions are met**:
1. The client process was successfully launched and owned via a verified run token.
2. The runtime probe completed the handshake.
3. The specified cache state was verified before the update began.
4. The Addressables update completed without unhandled exceptions.
5. Every asserted asset in the Content Contract was instantiated in memory and its C# `System.Type` matched the contract definition.

### 2. `FAILED`
A tile is marked `FAILED` if:
* The client crashed or exited prematurely.
* An Addressables async operation handle completed with `AsyncOperationStatus.Failed`.
* A declared contract asset was missing from the candidate catalog.
* An instantiated asset suffered a type mismatch (e.g., expected `AudioClip`, received `TextAsset`).
* The client hung and exceeded the scenario's bounded timeout (default: 30 seconds).

### 3. `UNVERIFIED`
A tile is marked `UNVERIFIED` whenever:
* The client was run in **External Observation** mode (wire traffic captured, but internal memory unobservable).
* An injected fault was configured, but the client code path failed to exercise the fault boundary (e.g., the client did not request the URL that carried the fault rule).
* A platform target is not supported for local process instrumentation (e.g., Android devices without a connected adb socket bridge).

### 4. `CANCELLED`
A tile is marked `CANCELLED` when:
* A user clicks **"Cancel Run"** in the Rehearsal Room window.
* A headless CI runner catches a SIGINT / abort signal.
* **Partial evidence is always retained**: All tiles completed prior to cancellation remain recorded in the report, while in-progress and subsequent tiles transition safely to `CANCELLED`.

---

## The Denominator Rule

In release compatibility reporting, many tools compute coverage percentages by filtering out unexecuted scenarios:
$$\text{Flawed Score} = \frac{\text{Passed}}{\text{Passed} + \text{Failed}} \times 100$$

If 10 scenarios were configured, but 6 could not be verified, the flawed score would report $4/4 = 100\%$ green!

PatchProof strictly enforces the **Denominator Rule**:
$$\text{PatchProof Truth Score} = \frac{\text{Verified Passed}}{\text{Total Declared Matrix Cells}} \times 100$$

Unverified, cancelled, and skipped cells **remain in the denominator**. 

If 7 out of 10 cells passed, and 3 are unverified:
* Total Score: **70% Verified**.
* CI Gate: **Blocked (Exit Code 3)**.
* Status Badge: **AMBER / BLOCKED**.

> ### Pro Tip: CI Promotion Integrity
> The headless CI runner (`PatchProofHeadlessRunner`) will **never exit with code 0** if a single tile in the matrix is marked `UNVERIFIED`. A release cannot be promoted to production on missing evidence.
