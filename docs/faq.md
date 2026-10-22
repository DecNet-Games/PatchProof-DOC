---
layout: default
title: Frequently Asked Questions
nav_order: 12
permalink: /docs/faq/
has_children: false
---

# Frequently Asked Questions (FAQ)
{: .no_toc }

Answers to common questions regarding architecture, cloud compatibility, and production integration.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 1. General & Architecture

### Is PatchProof a Content Delivery Network (CDN)?
**No.** PatchProof is an **in-Editor release staging laboratory**. It runs an embedded HTTP server on `127.0.0.1` inside your local Unity Editor process to simulate cloud deliveries over local loopback. It never hosts public assets and never routes traffic over the internet.

### Does PatchProof connect to our live production CDN?
**Never.** PatchProof is completely disconnected from your cloud infrastructure. It tests releases locally before you upload bundles to AWS S3, Google Cloud Storage, Cloudflare R2, or Unity Cloud Content Delivery (CCD). It protects your live players from broken catalogs by catching errors in staging.

### What Unity versions are officially supported?
PatchProof is developed and verified on **Unity 6000.5.4f1 LTS** with **Addressables 2.9.1**. While core C# staging logic may compile on earlier versions, Unity 6 LTS is the exact tested and supported runtime combination.

---

## 2. Testing & Verification

### What is the difference between an Instrumented Binary and a Release Twin?
* An **Instrumented Binary** is compiled from your current project with the `PatchProofProbe` component embedded in the bootstrap scene.
* A **Release Twin** is built from an archived Git tag representing a release shipped months ago, with only the probe package added to facilitate loopback control.

### Why does my CI build fail with Exit Code 3 when there are no errors?
Exit Code 3 means **Unverified Evidence Gate**. PatchProof refused to promote the release because one or more scenario tiles in your matrix were marked `UNVERIFIED`. This happens if you test in `ExternalObservation` mode (where internal memory cannot be asserted) or if an injected fault rule was configured but never triggered. PatchProof requires positive proof of success before granting a promotion pass.

### Can I run rehearsals on Android devices?
**No.** Android is **configuration only — NOT VERIFIED**. While Android manifest schemas and pairing settings exist in the codebase, physical on-device execution has not been verified. **Windows Standalone (x86_64)** is the tested and verified runtime rehearsal target.

---

## 3. Storage & Sandboxing

### Will PatchProof wipe my Editor's personal asset cache?
**Never.** Every rehearsal run generates a cryptographically unique `runToken` (e.g., `run_9a8b7c6d`). All cache operations are strictly confined to `PatchProof_Runs/{runToken}/Cache/`. Your Editor's personal cache in `AppData` and your project's `Library/` folder are completely untouched.

### Can I run multiple rehearsals concurrently on the same machine?
**Yes.** Because every run uses an isolated folder and can be configured with an independent port for its staging origin and probe server, multiple instances or parallel CI workers do not interfere with one another.

---

## 4. Licensing & Purchase

### Where can I acquire PatchProof?
PatchProof is available exclusively on the **[Unity Asset Store](https://assetstore.unity.com/packages/slug/421018)** (Short link: **[u3d.as/4g0P](https://u3d.as/4g0P)**). It is a commercial asset and is not distributed as open-source software.
