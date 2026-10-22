---
layout: default
title: CI/CD Pipeline Integration
parent: Workflows
nav_order: 5
permalink: /docs/workflows/ci-pipeline/
---

# CI/CD Pipeline Integration
{: .no_toc }

Complete guide to wiring automated headless rehearsals into GitHub Actions, GitLab CI, and Jenkins.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Automated Release Gates Matter

Manual testing in the Editor is essential during development, but release governance requires automated enforcement. By integrating PatchProof into your CI/CD pipeline:
* Every release candidate build automatically executes against historical client binaries.
* Pull requests that break content contracts are blocked before merging.
* Executive HTML dossiers are archived as build artifacts for compliance.

---

## Batchmode Architecture

PatchProof's continuous integration entry point is located in [`PatchProofHeadlessRunner.cs`](file:///c:/Users/anshw/Projects/Unity-Assets/Decnet_Unity_5_Flagships_15_Standalone_Docs/04_PatchProof_Addressables_Lab/PatchProof/Assets/Decnet/PatchProof/Editor/CI/PatchProofHeadlessRunner.cs).

When invoked from the command line, it parses arguments, initializes the headless matrix runner, and maps the results to process exit codes:

```
+-----------------------------------------------------------------------------+
|                        PatchProof Headless CI Runner                        |
+-----------------------------------------------------------------------------+
  |-- Input: -baselineRelease "v1.0.0" -candidateRelease "v2.0.0"
  |-- Input: -contractName "ProductionCoreContract"
  |-- Execution: Executes all 7 Matrix Scenarios via Loopback Origin
  |-- Output: Artifacts/rehearsal_report.html & rehearsal_report.json
  |-- Exit Code:
        * 0: Clean Pass (100% Verified)
        * 1: Rehearsal Failure (Contract Broken, Crash, Timeout)
        * 2: Configuration Error (Missing Files, Bad Flags)
        * 3: Unverified Cells Gate (Refuses to Promotes on Incomplete Data)
+-----------------------------------------------------------------------------+
```

---

## GitHub Actions Example

```yaml
name: Release Gate Rehearsal

on:
  push:
    tags:
      - 'v*'

jobs:
  run-patchproof-gate:
    runs-on: windows-2022
    steps:
      - name: Checkout Codebase
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Cache Unity Library
        uses: actions/cache@v4
        with:
          path: Library
          key: Library-${{ hashFiles('Packages/manifest.json') }}

      - name: Execute Headless Rehearsal
        shell: pwsh
        run: |
          $unityExe = "C:\Program Files\Unity\Hub\Editor\6000.5.4f1\Editor\Unity.exe"
          
          & $unityExe -batchmode -quit `
            -projectPath . `
            -executeMethod Decnet.PatchProof.Editor.PatchProofHeadlessRunner.RunHeadlessRehearsal `
            -baselineRelease "v1.0.0" `
            -candidateRelease "v2.0.0" `
            -contractName "CoreGameContract" `
            -reportPath "Artifacts/Report.html" `
            -jsonReportPath "Artifacts/Report.json" `
            -logFile "Logs/headless_run.log"
          
          if ($LASTEXITCODE -ne 0) {
            Write-Error "PatchProof Gate Failed with Exit Code: $LASTEXITCODE"
            exit $LASTEXITCODE
          }

      - name: Archive Rehearsal Dossier
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: PatchProof-Release-Dossier
          path: Artifacts/
```

---

## GitLab CI Example

```yaml
stages:
  - build
  - rehearsal
  - deploy

rehearse_update:
  stage: rehearsal
  tags:
    - windows
    - unity-6
  script:
    - '& "C:\Program Files\Unity\Hub\Editor\6000.5.4f1\Editor\Unity.exe" -batchmode -quit -projectPath . -executeMethod Decnet.PatchProof.Editor.PatchProofHeadlessRunner.RunHeadlessRehearsal -baselineRelease "v1.0.0" -candidateRelease "v2.0.0" -contractName "CoreGameContract" -reportPath "Artifacts/Report.html" -jsonReportPath "Artifacts/Report.json" -logFile "Logs/headless_run.log"'
  artifacts:
    when: always
    paths:
      - Artifacts/
      - Logs/
    expire_in: 30 days
```

> ### Pro Tip: Archiving Repro-Packs on Failure
> If your CI pipeline fails with Exit Code 1, write a quick post-step that automatically uploads the transient `PatchProof_Runs/{token}/` folder. Developers can download the zip and replay the failure locally in minutes.
