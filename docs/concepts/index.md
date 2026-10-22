---
layout: default
title: Core Concepts
nav_order: 5
permalink: /docs/concepts/
has_children: true
---

# Core Concepts
{: .no_toc }

Fundamental architectural principles behind Addressables release staging, runtime probes, and honest evidence matrices.
{: .lead }

PatchProof approaches asset delivery verification differently than traditional unit testing frameworks. Rather than mocking Addressables APIs or testing individual prefabs in Editor isolation, PatchProof treats **content delivery as a state machine distributed across networks and storage caches**.

---

## Conceptual Sections

* **[The Rehearsal Model]({{ site.baseurl }}/docs/concepts/rehearsal-model/):** The 5-stage lifecycle (Register &rarr; Declare &rarr; Set Conditions &rarr; Execute &rarr; Evidence).
* **[Client Modes]({{ site.baseurl }}/docs/concepts/client-modes/):** Deep dive into Instrumented Binary, Release Twin, and External Observation execution modes.
* **[Evidence & Unverified States]({{ site.baseurl }}/docs/concepts/evidence-and-unverified/):** Why an empty green matrix is refused, and how partial denominators preserve truth.
* **[Cache States & Invariants]({{ site.baseurl }}/docs/concepts/cache-states/):** How Fresh, WarmOld, and MixedPartial cache sandboxes simulate real-world player devices.
* **[Byte-Boundary Faults]({{ site.baseurl }}/docs/concepts/byte-boundary-faults/):** The mechanics of injecting network faults at specific packet offsets.
* **[Semantic Content Contracts]({{ site.baseurl }}/docs/concepts/content-contracts/):** Beyond string keys — asserting type safety and schema compatibility across releases.
