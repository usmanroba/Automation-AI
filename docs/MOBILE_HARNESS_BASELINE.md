# Mobile Harness Baseline

**Review date:** 2026-09-24  
**Reviewed source commit:** `15177fcb12bbd2afc8e281a79fb8185bb5daa5b8`  
**Baseline branch:** `manus/automation-ai-baseline`  
**Baseline version:** `versionName 1.0.4`, `versionCode 5`  
**Review scope:** Static source/configuration review, baseline unit tests, and online debug build. No application code was executed on an Android device or emulator.

## Project identity and repository state

The source project is Mobile Harness. The target repository is [usmanroba/Automation-AI](https://github.com/usmanroba/Automation-AI), and the untouched upstream is [techjarves/Mobile-Harness](https://github.com/techjarves/Mobile-Harness). The target repository existed as a public but empty repository; it was not a fork and had no prior commit, branch, tag, or file content to preserve. The source was copied from upstream while retaining its Git history. This is a Git-based copy, not a native GitHub fork.

At the start of the audit, the clone was on upstream `main` at commit `15177fcb12bbd2afc8e281a79fb8185bb5daa5b8`, the `v1.0.4` tagged revision. The target remote is configured as `origin`; upstream is configured as `upstream`. No changes were pushed to upstream. The working tree was clean before this baseline report was added.

## Repository structure

The Android application is the single Gradle module `:app`. Root files include the README, MIT license, privacy policy, Gradle configuration, F-Droid metadata, build/install/update test scripts, and documentation. Native code lives under `app/src/main/cpp`. PRoot and libandroid-shmem are upstream Git submodules pinned in the parent repository, with talloc source included in the native build. Tests are JVM unit tests under `app/src/test`; no `androidTest` instrumentation suite or `.github/workflows` CI workflow was present in the inspected tree.

## Architecture

The Android host is implemented in Kotlin with Jetpack Compose UI and a `MainViewModel`. A foreground service manages long-running runtime setup and user-started coding tasks. A C/C++ JNI bridge starts a rootless Ubuntu 20.04 ARM64 userspace through PRoot. The runtime shares app-private storage with project workspaces and installs command-line development tools there. This is an Android app sandbox plus a userspace container arrangement, not a virtual machine or hardened isolation boundary.

Projects, chats, settings, provider selections, and conversation identifiers are persisted with app-private files and `SharedPreferences`; terminal history is stored in app-private JSON files. Project import/export uses Android’s document/content picker contracts. The manifest disables Android backup, disallows general cleartext traffic, and declares Internet/network-state, notification, foreground-service/special-use, wake-lock, and package-install-request permissions.

The embedded Preview uses an Android `WebView` with JavaScript and DOM storage enabled for local development. Its URL validator limits navigation and requests to loopback hosts (`localhost`, `127.0.0.1`, and normalized `0.0.0.0`) and blocks external navigation and requests. The view reports load progress. No separate WebView console telemetry implementation was found.

## AI and local runtime

The installed coding agents are Claude Code, DeepSeek Harness, and Google’s Antigravity CLI. Provider choices include Anthropic, OpenRouter, DeepSeek, Kimi, OpenCode Zen, NVIDIA NIM, and a configurable custom API. Protocols include Anthropic-compatible and OpenAI-compatible gateway formats. Provider API keys are encrypted with AES-GCM using a key held in Android Keystore. Antigravity authentication is delegated to its CLI; the Android app records account state but the inspected code says it does not read or copy the CLI’s OAuth tokens.

No first-party local model inference engine or bundled model was found. A custom endpoint can be configured, but the repository does not establish that an on-device inference service is included. Local providers should not be described as a verified built-in feature.

## Terminal, files, Git, and GitHub

The terminal executes user/agent commands in the PRoot guest, streams process output, supports cancellation, records per-project history, and applies a limited blocklist to selected high-risk command patterns. This is not a general-purpose command authorization boundary. The README warns that Antigravity runs with `--dangerously-skip-permissions`, which allows its tools to act without individual app approval prompts; use it only with trusted projects and prompts.

Project files are browsed and edited inside private workspaces. The app supports user-selected content import/export, project archives, and generated Android APK installation through Android’s system installer. Git operations are available through the guest Git CLI and terminal. Public repository clone is implemented in the app; authenticated GitHub clone and login use GitHub CLI. GitHub client code also requests repository/account information. The inspected app does not implement a general in-app GitHub Releases publishing workflow.

## Build configuration

| Setting | Verified value |
|---|---|
| Gradle wrapper | Gradle 8.14 |
| Android Gradle Plugin | 8.13.2 |
| Kotlin Android / Compose plugins | 2.2.21 |
| Java/Kotlin bytecode target | Java 17 |
| Compile SDK | 36 |
| Target SDK | 28 for the direct online/offline builds; 36 when `playBuild=true` |
| Minimum SDK | 28 (Android 9) |
| NDK | 26.1.10909125 by default |
| Native build | CMake 3.22.1; ARM64 `arm64-v8a` only |
| Application ID / namespace | `com.jarves.mh` |
| Baseline version | `1.0.4` / code `5` |
| Variants | `online` and `offline` runtime-delivery flavors; debug and release build types |

The release signing configuration is conditional on four `MH_UPLOAD_*` environment variables. The repository’s Play release script instead expects a macOS Keychain and a default keystore path under `/Users/jarves`; neither corresponds to this Linux environment.

## Baseline build and test results

The baseline command was:

```text
./gradlew testOnlineDebugUnitTest testOfflineDebugUnitTest assembleOnlineDebug --console=plain
```

**Build status: PASS.** Gradle completed successfully. The online debug APK was generated at `app/build/outputs/apk/online/debug/app-online-debug.apk` (62,665,580 bytes). This is a debug build signed with the debug certificate, not a release-signed production APK. The C/C++ build emitted existing warnings, including deprecated `mktemp` usage and array-parameter `sizeof` warnings. Gradle also warned that the Kotlin `jvmTarget` configuration is deprecated and reported an Android SDK XML-version compatibility warning. The test/build report records actual outcomes; this baseline report does not claim a warning-free build.

| Test task | Result |
|---|---|
| `testOnlineDebugUnitTest` | PASS — 54 tests; 0 failures, 0 errors |
| `testOfflineDebugUnitTest` | PASS — 54 tests; 0 failures, 0 errors |
| Instrumentation / device install test | NOT RUN — no Android device or emulator was available |
| Baseline lint | Run separately; see final validation record |
| Dedicated static-analysis/security CI | NOT FOUND in repository |

## GitHub Actions and releases

No GitHub Actions workflow files were present. The README references upstream v1.0.4 artifacts. In-app updating downloads a manifest, validates package and version, checks the downloaded APK’s SHA-256 and compares its signing certificate with the installed app before invoking Android’s installer. No production release or direct download URL is established by this baseline audit.

## Security findings

The manifest sets `allowBackup=false` and cleartext networking is disallowed except for explicitly permitted loopback endpoints. API secrets use Android Keystore-backed encryption. Runtime downloads use checksum verification where implemented. These are positive controls, not proof of complete security.

The PRoot guest runs user-supplied and AI-generated shell commands and is not a hardened sandbox. The in-app destructive-command blocklist is narrow and can be bypassed by alternate command forms. Antigravity currently uses a skip-permissions mode. Runtime/package setup downloads software from upstream distribution endpoints and configured package repositories. A repository text scan for common credential patterns found no matching file paths; that scan is heuristic and is not a proof that no secrets exist.

## Privacy findings

`PRIVACY.md` describes a local-first design with prompts, files, terminal activity, attachments, and diagnostics stored in app-private storage. User-selected AI content is sent to the configured provider. Runtime setup contacts software distribution and package services. The policy states there are no advertising or analytics SDKs and no first-party account service. Provider and third-party retention are governed by their own terms. This review did not perform network packet capture or independently validate every policy claim.

## Known limitations and proposed improvements

Verified limitations include ARM64-only support, no built-in local inference engine, no device/emulator integration test in this environment, no repository CI workflow, and no production signing credentials configured in this workspace. These are observations, not new functionality requests. Any later changes should be labeled **PROPOSED** until approved and implemented.

## References

[1]: https://github.com/techjarves/Mobile-Harness/tree/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8 "Inspected upstream source commit"
[2]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/build.gradle.kts "Android build and signing configuration"
[3]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/src/main/AndroidManifest.xml "Android manifest and permissions"
[4]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/src/main/java/com/jarves/mh/ui/PocketDevApp.kt "Compose UI and local preview"
[5]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/src/main/java/com/jarves/mh/runtime/RuntimeInstaller.kt "PRoot runtime, checksums, and software installation"
[6]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/src/main/java/com/jarves/mh/data/ApiKeyVault.kt "Android Keystore API-key vault"
[7]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/app/src/main/java/com/jarves/mh/ui/MainViewModel.kt "Project execution, Git clone, and terminal handling"
[8]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/PRIVACY.md "Privacy policy"
[9]: https://github.com/techjarves/Mobile-Harness/blob/15177fcb12bbd2afc8e281a79fb8185bb5daa5b8/README.md "README and security warnings"
