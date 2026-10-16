# AE&D — DiPlay self-hosted compilation

**Date:** 2026-10-16  
**State:** EXPERIMENTAL — workflow committed for review; execution and on-vehicle validation pending.  
**Project:** DiPlay-AEND (fork of [shihabal3amri/DiPlay](https://github.com/shihabal3amri/DiPlay)).  
**CI configuration revision:** v0.1.0-alpha (workflow only; no application modifications).  
**App baseline:** upstream DiPlay v0.2.20, commit `3c95c936859e8a9153f0040caf2337adddb9e3e4`.  
**Target:** Lexus IS200 XE10, F.E.N.I.C.E. Android 9 MTK (not yet tested).

## Purpose and source-version discrepancy

Provide a reproducible, manual, source-only debug APK build using an AE&D self-hosted runner, without changing DiPlay application code or including CarPlay authentication material.

At initial inspection on 2026-10-16, fork `main` was at commit `1b1ac5f27986ce83b6d17fdd8c4d1be852fbbb70`, **38 commits after** the official v0.2.20 release commit, while still reporting `versionName = "0.2.20"`. These are distinct source revisions and must not be treated as equivalent. The workflow defaults to the **exact upstream release commit**. To test a newer or modified revision, explicitly change the **source_ref** input to its full commit SHA or chosen branch.

The release tag need not exist inside the fork; the referenced Git commit is present in its inherited history.

## Self-hosted environment

Previously known configuration, **not independently verified live**:
- Runner: `AEND-MacMini2011` on Debian 13 x86_64.
- Runner group: `aend build servers`.
- Runner labels: `self-hosted`, `Linux`, `X64`, `android`.

Required build environment (upstream [docs/BUILD.md](https://github.com/shihabal3amri/DiPlay/blob/v0.2.20/docs/BUILD.md)):
- Temurin JDK 25.
- Gradle 9.5.0 via project's wrapper.
- Android Platform 37, Build Tools 36.0.0, Platform Tools.
- Android NDK 28.2.13676358.

GitHub Actions installs/selects these components where supported. It may download dependencies on first run; allow sufficient disk space, networking, and the runner user permission to install SDK packages.

## Workflow

Path: `.github/workflows/build-diplay-aend.yml`.

1. Manual trigger only (`workflow_dispatch`). No automatic self-hosted execution on pushes or untrusted pull requests.
2. `source_ref` input defaults to the exact v0.2.20 commit; an explicit SHA or branch may be selected for future revisions.
3. Minimal `contents: read` permission, and checkout with `persist-credentials: false`.
4. Run upstream `scripts/check_public_tree.py` to reject credential files committed to the public tree.
5. Set up JDK, Android SDK/NDK and Gradle using documented actions.
6. Build only `:mobile:assembleDebug` with at most two Gradle workers.
7. Upload `mobile-debug.apk` together with resolved commit, environment and SHA-256 checksum as a GitHub Actions artifact for 30 days.

The source debug APK uses Android package `com.shihab.diplay.hudtest`. It is distinct from the release package `com.shihab.diplay` and the existing ZLink installation.

**Important:** GitHub requires a `workflow_dispatch` workflow file to exist on the **default branch** before the manual workflow becomes runnable. The proposed change is kept in a review branch and should be merged by an authorized maintainer after inspection. No build can be claimed until a workflow run completes successfully.

## GitHub activation

1. Review and merge the proposed CI pull request into `main`.
2. Under organization **Settings → Actions → Runner groups**, allow `aend-tech/DiPlay-AEND` in `aend build servers` if it is restricted.
3. Verify the runner is **online**, has the four labels listed above, and has access to the repo.
4. Open **DiPlay-AEND → Actions → AE&D DiPlay — Self-hosted Android Debug Build**.
5. Click **Run workflow**. Leave `source_ref` at the v0.2.20 SHA for the first run.
6. Download artifact `DiPlay-AEND-source-debug-<run_number>`. Keep `environment.txt` and the `.sha256` alongside the APK.
7. Validate installation on the Android 9 MTK head unit separately from ZLink; do not disable or remove the existing projection app.

## Authentication and security limitations

- **Source-only debug APK does not include CarPlay accessory identity.** Compiling successfully does not make the APK a functioning standalone CarPlay receiver. Upstream requires legitimate external authentication assets in its separate standalone build path.
- Do not upload identity private keys, MFi-related credentials, signing keys, or Android keystores to the fork or CI artifacts.
- The original DiPlay published preview uses an experimental identity not certified by Apple, with uncertain future acceptance and distribution rights.
- Executing third-party code on a persistent self-hosted runner carries a security risk. Manual dispatch reduces accidental execution but **does not isolate a malicious or compromised checkout**. For stronger isolation, use an ephemeral runner or controlled VM with no repository secrets and limited privileges.
- The upstream `.github/workflows/android.yml` remains unchanged and may continue running on GitHub-hosted runners on normal pushes/PRs; this addition does not disable it.
- Preserve copyright notices and verify GPL-3.0, AGPL-3.0 and third-party licenses before publishing a modified APK commercially.

## Test matrix and evidence

| Test | State | Evidence |
| --- | --- | --- |
| Upstream release SHA resolved | CONFIRMED BY GITHUB | `3c95c936859e8a9153f0040caf2337adddb9e3e4` |
| Fork `main` comparison | CONFIRMED BY GITHUB | 38 commits after tag on 2026-10-16 |
| Workflow file created | Pending verification | GitHub content check |
| Runner group accessibility | NOT VALIDATED | Organization Actions settings |
| Source compilation | NOT RUN | Future workflow logs |
| Debug APK installation | NOT RUN | Android 9 MTK |
| USB and wireless CarPlay | NOT RUN | Authentication and platform dependency |
| V.E.N.I.C.E. integration | NOT IMPLEMENTED | Future revision |

## Future development

Keep application integrations in separate feature branches, with new revisions and release notes: V.E.N.I.C.E. UART bridge, vehicle speed source, BEAN lighting to appearance mapping, multimedia controls and custom CarPlay side panel. The technical integration evidence belongs in the canonical vehicle repository `anyzunstudios/TOY-XE10-VNC`; derivative DiPlay source stays with its own original licensing requirements.

## Change history

### CI v0.1.0-alpha — 2026-10-16

- **Added:** manual self-hosted build workflow, reproducible ref input, SHA-256 and environment metadata artifacts.
- **Unchanged:** DiPlay application source, `aend-ci`, existing upstream GitHub workflow, ZLink and all vehicle firmware.
- **Reason:** allow controlled builds and differential testing of DiPlay on AE&D hardware.
- **Pending:** reviewer merge; access to runner group; actual CI compilation; installation and hardware validation.
