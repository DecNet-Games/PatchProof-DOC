---
layout: default
title: API Reference
nav_order: 9
permalink: /docs/api/
has_children: true
---

# Public API Reference
{: .no_toc }

Comprehensive technical reference for PatchProof's programmatic orchestration, probe protocol, and reporting APIs.
{: .lead }

While most day-to-day operations take place inside the **Rehearsal Room** Editor window, PatchProof provides a rich public C# API for custom automation, bespoke test harnesses, and studio build pipelines.

---

## API Modules

* **[`RehearsalRunner`]({{ site.baseurl }}/docs/api/rehearsal-runner/):** The core orchestration engine for executing matrices, dispatching scenarios, and monitoring real-time progress.
* **[`PatchProofProbe`]({{ site.baseurl }}/docs/api/probe-api/):** Runtime probe component API and loopback JSON-RPC request/response protocol.
* **[`ReleaseManifest`]({{ site.baseurl }}/docs/api/release-manifest/):** Data contracts for releases, catalog snapshots, and SHA-256 artifact entries.
* **[`ContentContract`]({{ site.baseurl }}/docs/api/content-contract/):** Defining programmatic contract invariants, expected types, and required components.
* **[`LocalStagingOrigin`]({{ site.baseurl }}/docs/api/staging-origin/):** Programmatic control of the embedded HTTP server, virtual routes, and fault profiles.
* **[`ReportGenerator`]({{ site.baseurl }}/docs/api/report-generator/):** Generating executive HTML reports and machine-readable JSON telemetry.
