---
layout: default
title: Troubleshooting
nav_order: 11
permalink: /docs/troubleshooting/
has_children: false
---

# Troubleshooting & Diagnostics
{: .no_toc }

Actionable solutions for port conflicts, process timeouts, cache locking, and probe disconnections.
{: .lead }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 1. Network & Port Issues

### Issue: "Staging Origin failed to start on port 8899"
* **Symptom:** The console logs `[LocalStagingOrigin] HttpListenerException: Access is denied` or `Address already in use`.
* **Root Cause:** Another process on your machine (e.g., an earlier unkilled rehearsal run or another local web server) is bound to port 8899.
* **Resolution:**
  1. Open `Tools > PatchProof > Origin & Fault Profiles`.
  2. Change **Server Port** to an open port (e.g., `8995`).
  3. Or run PowerShell to find and kill the lingering listener:
     ```powershell
     Get-NetTCPConnection -LocalPort 8899 | Select-Object OwningProcess
     Stop-Process -Id <OwningProcessId> -Force
     ```

---

## 2. Process & Probe Issues

### Issue: "Probe Handshake Timed Out (Port 8898)"
* **Symptom:** The Rehearsal Room launches the standalone player, but the progress bar stalls for 30 seconds and fails with `Operation timed out on Unity main thread`.
* **Root Cause:** The standalone player crashed during engine initialization, was blocked by Windows Defender Firewall, or did not load the scene containing `PatchProofProbe`.
* **Resolution:**
  1. Check Windows Firewall: Ensure `Allow local loopback connections for private networks` is enabled for the player executable.
  2. Verify Scene Hierarchy: Confirm that `PatchProofProbe` is attached to a GameObject in scene index 0 (the starting scene in Build Settings).
  3. Inspect Player Log: Open `PatchProof_Runs/{runToken}/Logs/client_player.log` to check for early engine boot crashes or missing graphics drivers.

---

## 3. Cache & Filesystem Issues

### Issue: "IOException: The process cannot access the file because it is being used by another process"
* **Symptom:** Clearing the run cache fails when attempting to delete a `.bundle` file.
* **Root Cause:** The standalone player process is still running in the background and holding an active file handle on the archive in `UnityEngine.Caching`.
* **Resolution:**
  1. PatchProof's `ProcessManager` will automatically attempt to kill the owned PID.
  2. If manually locked, navigate to `Tools > PatchProof > Rehearsal Room` and click **"Force Kill All Owned Processes"**.
  3. Check Windows Task Manager for orphaned instances of your game executable and terminate them.

---

## 4. Addressables Runtime Exceptions

### Issue: "RemoteProviderException: Unable to load asset bundle from 127.0.0.1:8899"
* **Symptom:** Addressables logs a 404 or connection failure during `UpdateCatalogs`.
* **Root Cause:** The virtual route mapping in the staging origin does not match the URL prefix embedded in your Addressables profile.
* **Resolution:**
  1. Check `Window > Asset Management > Addressables > Profiles`.
  2. Verify that **Remote.LoadPath** is configured as `http://127.0.0.1:8899/content/{version}/`.
  3. Ensure that your Release Manifest has `contentDirectory` pointing to the exact folder where the `.bundle` files were output.

---

## 5. Headless CI Issues

### Issue: "CI Runner exits with Exit Code 3"
* **Symptom:** Your automated build pipeline fails and logs `PatchProof CI Rehearsal Gate: Exit code 3`.
* **Root Cause:** The scenario matrix executed without crashes, but contains **Unverified Cells** (e.g., a fault profile rule was not exercised, or client mode was set to `ExternalObservation`).
* **Resolution:**
  1. Open the exported `rehearsal_report.html` from build artifacts.
  2. Look for amber `UNVERIFIED` tiles in the matrix.
  3. Ensure the test binary has the active probe attached (`InstrumentedBinary` mode).
  4. Ensure your fault rules match actual requested bundle URLs so the fault triggers and gets verified.
