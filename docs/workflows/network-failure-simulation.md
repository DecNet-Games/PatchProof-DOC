---
layout: default
title: Network Failure Simulation
parent: Workflows
nav_order: 3
permalink: /docs/workflows/network-failure-simulation/
---

# Network Failure & Drop Simulation
{: .no_toc }

Stress-testing download resumability, retry state machines, and graceful failure handling under severe network loss.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Hostile Reality of Player Networks

In development environments with high-speed fiber internet, Addressables downloads almost never fail. In the real world, your players download updates:
* On crowded cellular towers with 800 ms latency spikes.
* On subway trains passing through tunnels with total socket resets.
* Behind restrictive hotel or airport captive portals that intercept HTTP streams.

This workflow explains how to use PatchProof's **Fault Profiles** to harden your game's networking code.

---

## Step 1: Crafting the "Subway Tunnel" Fault Profile

1. Open `Tools > PatchProof > Origin & Fault Profiles`.
2. Click **"New Fault Profile"** and name it `SubwayTunnel_Drop`.
3. Configure **Rule 1 (Socket Reset)**:
   * **Fault Type:** `ConnectionDrop`
   * **URL Pattern:** `.*_assets_.*\.bundle` (Targets primary gameplay bundles).
   * **Trigger Byte Offset:** `262144` (Drops the connection after exactly 256 KB transferred).
4. Configure **Rule 2 (High Latency Retry)**:
   * **Fault Type:** `LatencyDelay`
   * **URL Pattern:** `.*`
   * **Delay Milliseconds:** `1500` (Forces all subsequent requests to wait 1.5 seconds).
5. Click **"Save Fault Profile"**.

---

## Step 2: Running the Rehearsal

1. Open `Tools > PatchProof > Rehearsal Room`.
2. Set your candidate update pairing.
3. In the **Active Fault Profile** dropdown, select `SubwayTunnel_Drop`.
4. Click **"Run Selected Scenario"** (targeting the `FreshCache_ConnectionDrop` tile).

### What You Should Observe in the Client Binary:
1. The client begins downloading the bundle.
2. The download progress bar reaches ~15% and stops.
3. The local origin closes the TCP socket.
4. **Good Client Behavior:**
   * The client catches the `ConnectionError`.
   * A non-blocking retry indicator appears: *"Connection interrupted. Retrying in 3... 2... 1..."*
   * The client initiates a new request with an HTTP `Range: bytes=262144-` header.
   * The download finishes, and the asset loads cleanly.
5. **Bad Client Behavior (Bug Discovered!):**
   * The game freezes completely on the main thread.
   * An unhandled `NullReferenceException` is logged.
   * The client crashes to desktop or enters an infinite reload loop.

---

## Step 3: Capturing Evidence with a Repro-Pack

If your game fails the test:
1. Immediately click **"Export Repro-Pack"** on the failed tile in the Rehearsal Room.
2. The exported archive contains the exact `SubwayTunnel_Drop` profile and the request trace showing the socket abort at byte 262,144.
3. Hand the package to your gameplay networking engineer to implement proper retry backoff logic.
