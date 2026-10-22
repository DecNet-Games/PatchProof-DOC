# PatchProof Documentation Repository

This repository contains the complete documentation site for **PatchProof — Addressables Release Rehearsal Lab for Unity**, hosted via GitHub Pages using the `just-the-docs` theme.

* **Live Documentation:** [https://decnet-games.github.io/PatchProof-DOC/](https://decnet-games.github.io/PatchProof-DOC/)
* **Unity Asset Store Package:** [https://assetstore.unity.com/packages/slug/421018](https://assetstore.unity.com/packages/slug/421018)
* **Short Link:** [https://u3d.as/4g0P](https://u3d.as/4g0P)

---

## Technical Specifications & Verification Fence

* Developed and tested on **Unity 6000.5.4f1 LTS** with **Addressables 2.9.1**.
* Windows Standalone (x86_64) is the tested and verified runtime rehearsal path.
* Android is **configuration only — NOT VERIFIED** (no device/platform run).
* Ships with **31 automated tests** (29 EditMode tests independently passing 29/29; 2 PlayMode tests per project evidence only).
* Spec acceptance coverage: **12 of 12 spec test areas verified** (11 via EditMode suite, 1 via PlayMode project evidence).
* Headless CI runner self-reports **Exit code 3 (unverified fault cells)** on incomplete evidence.
* PatchProof is **NOT a CDN** and never touches production endpoints.

---

Copyright &copy; 2026 Decnet Games. All rights reserved.
