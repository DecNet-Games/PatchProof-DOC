---
layout: default
title: Headless CI Runner
parent: Features Overview
nav_order: 13
permalink: /docs/features/ci-runner/
---

# Headless CI Runner
{: .no_toc }

Automating Addressables update rehearsals in headless CI pipelines with honest exit codes.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Moving Rehearsals into Continuous Delivery

While the in-Editor **Rehearsal Room** is ideal for interactive debugging, release governance requires running the exact same verification matrix in **Continuous Integration (CI)** pipelines on every pull request and release build.

PatchProof provides a dedicated batchmode entry point in [`PatchProofHeadlessRunner.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/CI/PatchProofHeadlessRunner.cs).

---

## Batchmode Command Syntax

Invoke Unity in batchmode pointing to PatchProof's static execution method:

```powershell
& "C:\Program Files\Unity\Hub\Editor\6000.5.4f1\Editor\Unity.exe" `
  -batchmode `
  -quit `
  -projectPath "C:\MyGameProject" `
  -executeMethod "Decnet.PatchProof.Editor.PatchProofHeadlessRunner.RunHeadlessRehearsal" `
  -baselineRelease "v1.0.0" `
  -candidateRelease "v2.0.0" `
  -contractName "CoreGameContract" `
  -reportPath "Artifacts/PatchProof_Report.html" `
  -jsonReportPath "Artifacts/PatchProof_Report.json" `
  -logFile "Logs/headless_rehearsal.log"
```

---

## Strict Exit Code Contract

Continuous delivery pipelines rely on standardized process exit codes to pass or fail build stages. PatchProof enforces an explicit 4-way contract:

| Exit Code | Semantic Meaning | CI Pipeline Action |
|:---:|:---|:---|
| **`0`** | **CLEAN PROCEED**<br>Every matrix scenario tile passed with verified runtime probe memory assertions. Zero contract violations, zero unhandled errors. | Promote release to staging CDN / trigger deploy pipeline. |
| **`1`** | **REHEARSAL FAILURE**<br>One or more scenario tiles failed (crash, contract missing, type mismatch, timeout unhandled). | **Block build.** Publish failure summary to PR / Slack. |
| **`2`** | **CONFIGURATION ERROR**<br>Bad command-line flags, missing baseline manifest, corrupted catalog path, or port conflicts. | **Block build.** Alert DevOps team of configuration drift. |
| **`3`** | **UNVERIFIED EVIDENCE GATE**<br>The run completed without crashes, but one or more cells were **unverified** (e.g., run in External Observation mode, or a fault rule was unexercised). | **Block promotion.** Prevent shipping releases on incomplete evidence. |

---

## GitHub Actions Workflow Integration

Here is a production-ready GitHub Actions step using PatchProof:

```yaml
name: Addressables Release Gate

on:
  push:
    branches: [ release/* ]

jobs:
  rehearse-update:
    runs-on: windows-2022
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Build Addressables Candidate Content
        run: |
          & "C:\Program Files\Unity\Editor\Unity.exe" `
            -batchmode -quit -projectPath . `
            -executeMethod AddressablesPlayerBuildProcessor.BuildAddressablesContent `
            -logFile Logs/build_addressables.log

      - name: Run PatchProof Rehearsal Matrix
        run: |
          & "C:\Program Files\Unity\Editor\Unity.exe" `
            -batchmode -quit -projectPath . `
            -executeMethod Decnet.PatchProof.Editor.PatchProofHeadlessRunner.RunHeadlessRehearsal `
            -baselineRelease "v1.0.0" `
            -candidateRelease "v2.0.0" `
            -contractName "CoreGameContract" `
            -reportPath "Reports/rehearsal_report.html" `
            -jsonReportPath "Reports/rehearsal_report.json" `
            -logFile Logs/headless_rehearsal.log
          
          # PowerShell preserves the native Unity exit code
          if ($LASTEXITCODE -ne 0) {
            Write-Error "PatchProof Rehearsal Gate Failed with Exit Code: $LASTEXITCODE"
            exit $LASTEXITCODE
          }

      - name: Upload Audit Dossier
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: patchproof-audit-report
          path: Reports/
```

> ### Pro Tip: Self-Reported Gate Honesty
> In our own continuous integration tests on clean candidate runs where network fault interception was intentionally skipped, PatchProof's headless runner self-reported **Exit Code 3 (unverified fault cells)**. It refuses to invent passing marks!
