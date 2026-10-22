---
layout: default
title: Features Overview
nav_order: 6
permalink: /docs/features/
has_children: true
---

# Feature Deep Dives
{: .no_toc }

Comprehensive architectural and usage documentation for every tool, engine, and window in PatchProof.
{: .lead }

PatchProof is composed of 13 modular subsystems engineered to provide end-to-end release safety for Unity Addressables projects.

---

## The Feature Suite

<div class="cards">
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/rehearsal-room/">1. Rehearsal Room</a></h3>
      <p>The primary Editor control room: environment banners, scenario selection, real-time matrix updates, and cancellation controls.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/release-registry/">2. Release Registry</a></h3>
      <p>Immutable release manifests, SHA-256 artifact hashing, and version pairing definitions.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/staging-origin/">3. Staging Origin Server</a></h3>
      <p>Embedded local HTTP origin, route mapping, URL rewriting, and zero-CDN loopback testing.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/fault-injection/">4. Fault Injection Engine</a></h3>
      <p>Simulating latency, abrupt socket drops, 404/500 errors, payload bit-flips, and truncation at byte boundaries.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/client-probe/">5. Runtime Client Probe</a></h3>
      <p>Lightweight MonoBehaviour probe, loopback HTTP command protocol, and in-memory contract evaluation.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/cache-profiles/">6. Cache Profiles & Isolation</a></h3>
      <p>Managing Fresh, WarmOld, and MixedPartial cache sandboxes with run-token isolation.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/coverage-matrix/">7. Coverage Matrix</a></h3>
      <p>Multi-dimensional scenario grid displaying real-time execution, timing, and promotion readiness.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/request-traces/">8. Request Traces & Telemetry</a></h3>
      <p>Timeline visualization of HTTP requests, status codes, wire bytes, and cache-hit metrics.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/catalog-and-contracts/">9. Catalogs & Contracts</a></h3>
      <p>Static catalog comparison, breaking change detection, and C# type schema compatibility checks.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/rollback-planner/">10. Rollback Planner</a></h3>
      <p>Computing dependency closures for previous versions and preventing missing-history deployment traps.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/repro-packs/">11. Repro-Packs</a></h3>
      <p>Exporting self-contained bug reproduction archives with exact catalogs, faults, and replay steps.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/reports/">12. Audit Reports & Checklists</a></h3>
      <p>Generating executive HTML reports, machine-readable JSON metrics, and promotion checklists.</p>
    </div>
  </div>
  <div class="card">
    <div class="card-content">
      <h3 class="card-title"><a href="{{ site.baseurl }}/docs/features/ci-runner/">13. Headless CI Runner</a></h3>
      <p>Command-line batchmode runner with automated exit codes (0 clean, 1 failure, 2 config error, 3 unverified).</p>
    </div>
  </div>
</div>
