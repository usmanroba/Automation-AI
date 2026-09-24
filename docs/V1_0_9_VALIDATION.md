# U&U v1.0.9 validation and release status

**Date:** 2026-09-24; **Branch:** `manus/automation-ai-v1.0.9`
**Scope:** User-facing Android rename to U&U, version update to 1.0.9, and in-app update-manifest routing to the target repository. Package ID, namespace, and upstream attribution remain unchanged.

## Identity checks

| Check | Result |
|---|---|
| Android launcher/app label | PASS — compiled APK metadata reports `U&U` |
| `versionName` | PASS — `1.0.9` |
| `versionCode` | PASS — `6`, sequential after upstream v1.0.4/code 5 |
| Package ID / namespace | PASS — retained as `com.jarves.mh` to preserve project continuity |
| In-app update channel | PASS — points to the `usmanroba/Automation-AI` release manifest; release asset is unavailable until signed publication |
| Runtime execution/features | Unchanged; product-name UI/prompts, version, and update-manifest configuration were updated |

## Build and tests

Command:

```text
./gradlew testOnlineDebugUnitTest testOfflineDebugUnitTest assembleOnlineDebug --console=plain
```

**Debug compile/package: PASS.** Both online and offline debug unit-test tasks pass: 54 tests each (108 across both flavors), zero failures and errors. The online debug APK includes the upstream shared AGY runtime bundle downloaded from `techjarves/Mobile-Harness` and verified against the official release SHA-256 `a659ab9188956fc4721ca86fb21b5118e0e489f47a5e02ae6b4f2fb423659d78`. The generated online debug APK is 104,544,878 bytes and its APK signature verifies under the Android **debug** certificate. This is a test artifact, **not** the distributable release APK.

Online lint: `./gradlew lintOnlineDebug --console=plain` **PASS** after the rename; report contains **0 errors, 67 warnings, 1 hint**. A detached baseline run also passed with the same **0 errors, 67 warnings, 1 hint**, so the rename introduced no lint-count increase.

No Android emulator/device was available; installation and launch testing were not run. The app source is ARM64-only; this environment did not run the APK on ARM64 hardware.

## Release packaging and signing

Command:

```text
./gradlew assembleOnlineRelease --console=plain
```

**RELEASE_BUILD_STATUS=PASS** — Gradle completed the online release-variant packaging task (`BUILD SUCCESSFUL`), producing `app/build/outputs/apk/online/release/app-online-release-unsigned.apk` (87,413,327 bytes). Compiled metadata checks pass: package `com.jarves.mh`, version name `1.0.9`, version code `6`, label `U&U`. The artifact is explicitly named `-unsigned.apk`. `apksigner verify --verbose --print-certs` reports `DOES NOT VERIFY` / `ERROR: Missing META-INF/MANIFEST.MF`; it is **not a signed release APK** and must not be distributed. No APK installation test was run.

The Gradle build emitted existing C/C++ warnings in bundled PRoot code (misleading indentation, missing return on a non-void path, and deprecated `mktemp`) and Kotlin/Java deprecation warnings (`ACTION_INSTALL_PACKAGE` and auto-mirrored Compose icons). The online release lint-vital task completed successfully. The repository release configuration only enables its upload signing config when all four `MH_UPLOAD_*` environment inputs are provided. No verified original release keystore/key is configured in this workspace.

A public upstream v1.0.4 online APK was downloaded read-only and verified against its GitHub asset SHA-256. Its signer certificate SHA-256 fingerprint is `d364b1edd80b955e7fe9d99edc4cc211ce723e461ded6a5160c6a2db3fdd7af1` (RSA 4096). The local debug APK has a different Android debug certificate. The upstream public certificate does not enable signing; the corresponding private key is needed to produce updates that Android accepts as upgrades to existing installations.

**APK_SIGNING_STATUS=BLOCKED. RELEASE_SIGNING_STATUS=BLOCKED.** Do not create or upload a release using a new invented key. A new key would not match the established upstream signer for the unchanged application ID and would prevent in-place upgrades for existing installs unless the existing release key/valid signing lineage is obtained. The next required input is the original authorized signing identity, configured securely through the project’s `MH_UPLOAD_*` inputs. Do not put the keystore, passwords, or private keys in Git.

**GITHUB_RELEASE_STATUS=NOT CREATED.** The release workflow requires a verified signed APK. No APK release asset or direct download URL is claimed.

## Validation distinctions

A debug APK being signed with the Android debug certificate does not satisfy release-signing requirements. Release optimization, package/version metadata, and test success do not establish that users can safely install or upgrade the build. No GitHub release should be published until a correctly signed release artifact has been generated and its signature/package/version have all been verified.
