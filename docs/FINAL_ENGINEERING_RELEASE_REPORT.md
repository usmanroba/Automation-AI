# MOBILE HARNESS → AUTOMATION-AI

# FINAL ENGINEERING & RELEASE REPORT

**Report date:** 2026-09-25  
**Prepared by:** Manus AI  
**Result:** Source audit, baseline, research repository, and U&U v1.0.9 release branch are complete. A distributable release is **blocked by the missing authorized signing key**. No GitHub release, tag, APK asset, or direct download URL was created.

## 1. GitHub account

**Username:** `usmanroba`  
**Authentication:** VERIFIED. The authenticated GitHub CLI account returned `usmanroba` during repository checks and immediately before the production branch push.

## 2. Repositories

**Production target:** [usmanroba/Automation-AI](https://github.com/usmanroba/Automation-AI) [1]  
**Source upstream:** [techjarves/Mobile-Harness](https://github.com/techjarves/Mobile-Harness) [2]  
**Production release branch:** [`manus/automation-ai-v1.0.9`](https://github.com/usmanroba/Automation-AI/tree/manus/automation-ai-v1.0.9) [3]  
**Research repository:** [usmanroba/researches](https://github.com/usmanroba/researches), private [5]

## 3. Production repository status

**Status:** EXISTING and VERIFIED. `Automation-AI` existed as a public, empty repository when inspected. GitHub reported `isFork=false` and no parent repository. It was not a native GitHub fork. The safe strategy was to preserve Mobile Harness’s existing Git history in a local clone, use `usmanroba/Automation-AI` as `origin`, and retain `techjarves/Mobile-Harness` as the read-only-use `upstream`. The baseline and release work was kept on dedicated branches rather than rewriting the upstream or force-pushing `main`.

## 4. Git status

**Origin:** VERIFIED — `https://github.com/usmanroba/Automation-AI`  
**Upstream:** VERIFIED — `https://github.com/techjarves/Mobile-Harness`  
**Current release branch:** `manus/automation-ai-v1.0.9`  
**Release source commit:** `05136cf7f4e1146d0380a287738aa76889c8b4e4`  
**Commit message:** `release: prepare U&U v1.0.9`  
**Push:** VERIFIED — the branch tip on GitHub matches the local commit.  
**Working tree:** CLEAN at final verification.

The baseline is separately recorded on [`manus/automation-ai-baseline`](https://github.com/usmanroba/Automation-AI/tree/manus/automation-ai-baseline), commit `f8fdb51` (`docs: establish Mobile Harness baseline`). GitHub’s current default branch is `manus/automation-ai-baseline`. The target currently has two remote branches: the baseline and this v1.0.9 release branch; it has no remote `main` branch. The baseline was the first branch pushed to this previously empty target, and no default-branch setting was changed during this work. No release-branch merge or pull request was created.

## 5. Fork/copy status

**Status:** VERIFIED — independent GitHub repository, not a native fork. `Automation-AI` contains the Mobile Harness source history copied from the upstream project. GitHub’s `parent` field was empty. The Android package and namespace remain `com.jarves.mh`, preserving the existing application identity needed for continuity with installed builds.

## 6. Clone status

**Status:** VERIFIED. The local clone is `/home/ubuntu/workspace/Automation-AI`. Its base source commit is upstream `main` at `15177fcb12bbd2afc8e281a79fb8185bb5daa5b8`, tagged `v1.0.4`. The original upstream was not modified. The two native submodules remain pinned to their inherited revisions.

## 7. Architecture audit

The app is a single Android Gradle module, `:app`, implemented in Kotlin with Jetpack Compose and a `MainViewModel`. A foreground service manages long-running user-started tasks. Native C/C++ JNI code launches a rootless ARM64 Ubuntu userspace through PRoot. This is an Android app sandbox plus userspace arrangement, not a virtual machine or a hardened security boundary.

Projects and conversations are held in app-private storage and preferences. User-selected document-picker flows support import and export. Terminal history is stored in app-private JSON files. The terminal executes commands in the PRoot guest and supports output streaming and cancellation. A limited command blocklist is not a general command authorization boundary.

The web preview is an Android WebView configured for local development. The reviewed validator restricts preview navigation and requests to loopback hosts and blocks external navigation and requests. JavaScript and DOM storage are enabled for the local preview.

AI coding agents found in source are Claude Code, DeepSeek Harness, and Antigravity CLI. Provider choices include Anthropic, OpenRouter, DeepSeek, Kimi, OpenCode Zen, NVIDIA NIM, and a custom endpoint. Anthropic-compatible and OpenAI-compatible protocols are represented in code. **NOT FOUND:** a first-party local inference engine or bundled model. A custom endpoint does not establish that an on-device model service is provided.

Provider keys are encrypted using AES-GCM with an Android Keystore-held key. Antigravity authentication is handled by its CLI; the inspected integration says the app does not read or copy its OAuth tokens. Git commands run through the guest CLI. Public clone is implemented in-app, while authenticated GitHub login uses GitHub’s device flow and the GitHub CLI. **NOT FOUND:** a general in-app GitHub Releases publishing workflow. The app has an update-manifest client which validates package identity, version, SHA-256 when provided, and signer equality before installation.

## 8. Android build environment

**Build status:** PASS for the final unit-test, online lint, online debug packaging, and online release-variant tasks.

**Final command:**

```text
./gradlew testOnlineDebugUnitTest testOfflineDebugUnitTest lintOnlineDebug assembleOnlineDebug assembleOnlineRelease --console=plain
```

The aggregate task completed with `BUILD SUCCESSFUL` after the final updater-manifest change. The environment used Gradle 8.14, Android Gradle Plugin 8.13.2, Kotlin 2.2.21, JDK 17, compile SDK 36, direct-build target SDK 28, minimum SDK 28, NDK 26.1.10909125, and CMake 3.22.1. The app is configured for ARM64 (`arm64-v8a`).

The release-variant Gradle packaging task passes. The resulting APK is unsigned, so this build status does not mean a production-ready signed APK was created.

## 9. Baseline and final tests

The baseline online and offline debug unit-test tasks each passed **54 tests, zero failures, zero errors**. The baseline online debug APK built successfully. No Android device or emulator was available, so instrumentation, installation, and launch tests were not run.

The final U&U v1.0.9 online and offline debug unit-test tasks also passed **54 tests each, zero failures, zero errors**. Final online debug lint passed with **0 errors, 67 warnings, and 1 hint**. A detached clean baseline lint run reported the same totals, so this change introduced no increase in lint counts. The release lint-vital task completed successfully. No separate security-analysis CI workflow or GitHub Actions workflow was present in the inspected repository.

## 10. Continuous integration

**GitHub Actions:** NOT FOUND — there were no workflow files under `.github/workflows` in the inspected source tree. No CI checks or security workflows were therefore available to report as passing.

## 11. Security findings

The manifest disallows general cleartext traffic, disables Android backup, and declares networking, notification, foreground-service, wake-lock, and package-install-request permissions. The Android runtime confines guest files to app-private areas through the app and PRoot setup, but PRoot is not hardened isolation. The README warns that Antigravity runs with `--dangerously-skip-permissions`, which permits agent tools to act without individual app approval prompts. Only trusted projects and prompts should be used with that mode.

The updater compares the APK signer with the installed app signer. This is an important release constraint: a newly invented signing key will not produce an in-place update for users of the existing app. A repository-path secret scan found no matching tracked secret files during the audit. No credentials or signing material were added to commits. This was a static source/configuration review, not a penetration test or device security assessment.

## 12. Privacy findings

The inspected policy describes projects, conversations, attachments, runtime files, terminal history, and diagnostics as local app-private data. Provider-request content is sent to the AI service selected by the user. Runtime setup downloads software from upstream distribution services. The inspected project did not include advertising or analytics SDKs or a first-party account backend. These are source-review findings and do not substitute for current vendor privacy terms.

The runtime bundle remains sourced from the upstream project’s shared runtime release. The U&U application update-manifest URL is routed to `usmanroba/Automation-AI`; until a signed release and manifest are published there, no update asset is available from that endpoint.

## 13. Baseline documentation

**File:** [`docs/MOBILE_HARNESS_BASELINE.md`](MOBILE_HARNESS_BASELINE.md)  
**Status:** CREATED, reviewed, committed, and pushed on `manus/automation-ai-baseline` at `f8fdb51`. It documents source architecture, build configuration, baseline tests, security/privacy findings, CI status, and limitations.

## 14. v1.0.9 application update

**Application name:** U&U  
**Version name:** `1.0.9`  
**Version code:** `6`  
**Application ID / namespace:** `com.jarves.mh`  
**Updater channel:** `https://github.com/usmanroba/Automation-AI/releases/latest/download/mobile-harness-update.json`

The Android launcher label was confirmed from compiled APK metadata. Product-name UI and agent guidance strings were updated without renaming package, class, resource, or internal workspace identifiers. Core runtime behavior was not intentionally changed.

## 15. v1.0.9 build artifacts

**Release build status:** PASS for release-variant packaging only.  
**Command:** `./gradlew assembleOnlineRelease --console=plain`  
**APK name:** `app-online-release-unsigned.apk`  
**APK path:** `app/build/outputs/apk/online/release/app-online-release-unsigned.apk`  
**Final APK size:** 87,413,327 bytes.

The latest online debug artifact is `app/build/outputs/apk/online/debug/app-online-debug.apk`, 104,544,878 bytes. It contains the shared AGY runtime bundle downloaded from the upstream `runtime-2026.09.4` release and checked against SHA-256 `a659ab9188956fc4721ca86fb21b5118e0e489f47a5e02ae6b4f2fb423659d78`. It is signed with the Android debug certificate, not a release certificate, and is not a distributable release APK.

## 16. APK signing

**APK signing status:** BLOCKED.  
**Release signature verification:** FAILS as expected for the unsigned artifact; `apksigner` reports `DOES NOT VERIFY` and `ERROR: Missing META-INF/MANIFEST.MF`.

No authorized upload keystore or private signing key was configured in this environment. The published upstream v1.0.4 APK was checked read-only; its signer certificate SHA-256 fingerprint is `d364b1edd80b955e7fe9d99edc4cc211ce723e461ded6a5160c6a2db3fdd7af1` (RSA 4096). The public certificate does not contain the private key needed to sign an upgrade-compatible APK. No substitute key was generated.

## 17. APK validation

Compiled metadata from both final online variants reports application ID `com.jarves.mh`, label `U&U`, version name `1.0.9`, and version code `6`. The online debug APK signature verifies under the debug key. The release-variant APK has no signature and cannot be installed as a production update. **Installation test:** NOT RUN — no Android device or emulator was available.

## 18. v1.0.9 test results

Both debug-flavor unit-test suites passed with 54 tests each and zero failures/errors. `lintOnlineDebug` passed with 0 errors, 67 warnings, and 1 hint; the clean baseline produced the same totals. Online debug packaging and online release-variant packaging completed successfully. The release-variant output is unsigned. No instrumentation test ran.

## 19. Git commit

**Release source commit:** `05136cf7f4e1146d0380a287738aa76889c8b4e4`  
**Commit message:** `release: prepare U&U v1.0.9`  
**Branch:** `manus/automation-ai-v1.0.9`  
**Push status:** VERIFIED on `origin`.  
**Pull request / merge:** NOT CREATED; `main` was not force-pushed or overwritten.

## 20. GitHub release

**Repository:** [usmanroba/Automation-AI](https://github.com/usmanroba/Automation-AI)  
**Requested tag:** `v1.0.9`  
**Requested release name:** `U&U v1.0.9`  
**GitHub release status:** NOT CREATED. The repository had no releases, and no `v1.0.9` tag exists. Creating a public release without a valid signed APK would misrepresent the artifact’s installability and upgrade compatibility.

## 21. Release APK asset

**Asset:** NOT UPLOADED.  
**Status:** BLOCKED by the missing authorized signing key. The unsigned local release-variant APK was not uploaded.

## 22. Direct APK download

**Status:** No verified direct APK download link exists. No link is provided or fabricated. A direct link can be reported only after a signed APK is uploaded to a verified GitHub release.

## 23. Files created

Production files created during this work:

- `docs/MOBILE_HARNESS_BASELINE.md`
- `docs/V1_0_9_VALIDATION.md`
- `docs/FINAL_ENGINEERING_RELEASE_REPORT.md` (this report)

The separate research commit added `.gitmodules` and the following files or submodule pointers. `README.md` and `project-index.md` were updated rather than created.

```text
agent-frameworks/README.md
agent-frameworks/additional-projects/README.md
autonomous-ai/README.md
autonomous-ai/additional-projects/README.md
autonomous-ai/additional-projects/agenticSeek
autonomous-ai/additional-projects/agenticseek-audit.md
autonomous-ai/additional-projects/agenticseek-metadata.md
autonomous-ai/additional-projects/gptme
autonomous-ai/additional-projects/gptme-audit.md
autonomous-ai/additional-projects/gptme-metadata.md
autonomous-ai/additional-projects/suna-exclusion.md
autonomous-ai/manus-open/README.md
autonomous-ai/open-operator
autonomous-ai/open-operator-audit.md
autonomous-ai/open-operator-metadata.md
autonomous-ai/openhands
autonomous-ai/openhands-audit.md
autonomous-ai/openhands-metadata.md
autonomous-ai/openmanus
autonomous-ai/openmanus-audit.md
autonomous-ai/openmanus-metadata.md
browser-agents/README.md
browser-agents/additional-projects/README.md
coding-agents/README.md
coding-agents/additional-projects/README.md
comparisons/README.md
comparisons/additional-candidates.md
comparisons/project-comparison.md
computer-use/README.md
computer-use/additional-projects/README.md
integration-proposals/README.md
integration-proposals/proposed-capabilities.md
license-audits/README.md
local-ai/README.md
local-ai/additional-projects/README.md
mcp-agents/README.md
mcp-agents/additional-projects/README.md
project-index.md
sandbox-technologies/README.md
sandbox-technologies/additional-projects/README.md
security-audits/README.md
```

## 24. Files modified

The v1.0.9 release commit modified the following existing production files:

```text
app/build.gradle.kts
app/src/main/java/com/jarves/mh/runtime/ClaudeRuntimeBridge.kt
app/src/main/java/com/jarves/mh/runtime/DshRuntimeBridge.kt
app/src/main/java/com/jarves/mh/runtime/RuntimeExecutionService.kt
app/src/main/java/com/jarves/mh/runtime/RuntimeInstaller.kt
app/src/main/java/com/jarves/mh/runtime/RuntimeSetupService.kt
app/src/main/java/com/jarves/mh/ui/MainViewModel.kt
app/src/main/java/com/jarves/mh/ui/PocketDevApp.kt
app/src/main/java/com/jarves/mh/ui/SettingsScreen.kt
app/src/main/java/com/jarves/mh/ui/SettingsScreenModern.kt
app/src/main/java/com/jarves/mh/ui/TerminalScreen.kt
app/src/main/java/com/jarves/mh/update/AppUpdater.kt
app/src/main/res/values/strings.xml
docs/MOBILE_HARNESS_BASELINE.md
```

The separate research commit modified `README.md` and `project-index.md` in the private research repository. The new release validation report and final report are listed above as created files.

## 25. Problems and blockers

**Authorized signing key unavailable.** The release build was attempted and successfully produced an unsigned online release APK. Its signer verification failed because it has no signing manifest. The existing upstream signer fingerprint was identified from the published v1.0.4 APK, but the corresponding private key is not included in public releases and was not present in the sandbox. Resolution requires the project owner to provide the already-authorized upload keystore through a secure channel and supply the configured `MH_UPLOAD_STORE_FILE`, `MH_UPLOAD_STORE_PASSWORD`, `MH_UPLOAD_KEY_ALIAS`, and `MH_UPLOAD_KEY_PASSWORD` inputs in the build environment. Do not paste secrets into chat or commit them.

**Device installation testing unavailable.** The environment has no connected Android device or emulator, and the app is ARM64-only. APK metadata and signatures were inspected on the build host, but installation and upgrade behavior were not tested. Resolution requires testing on a compatible ARM64 Android device after signing is restored.

**No published release asset or update manifest.** The in-app update channel now targets the Automation-AI release manifest, but the `v1.0.9` release and signed APK do not exist yet. The updater endpoint will not have a new version asset until the signing blocker is resolved and the release is published.

## 26. Proposed next steps

1. **PROPOSED:** Configure the existing authorized release keystore and passwords securely in the release environment. Do not create a new signing identity.
2. **PROPOSED:** Rebuild the online release APK and verify the APK signature, package ID, version, label, and checksum. Test installation and upgrade on an ARM64 Android device.
3. **PROPOSED:** After those checks pass, create the `v1.0.9` tag and `U&U v1.0.9` GitHub release. Upload the signed APK and the updater manifest, then verify the published asset and direct download URL.
4. **PROPOSED:** Consider a pull request or default-branch migration separately. No merge or default-branch setting change was performed as part of this work.

## Research repository result

The separate [private research repository](https://github.com/usmanroba/researches) is on `main` at commit `9a2685fd30529662101de2ac38c8f5e494c5541a` (`research: add autonomous AI project audits`). It contains six project reviews, five licensed source repositories pinned as independent submodules, license and security registers, comparison matrices, and explicitly proposed-only integration notes. The archived `manus-open` source was not cloned because no repository-level license was verified. Suna was not cloned because its Elastic License 2.0 is not OSI-approved open-source. None of the research code was executed or copied into the production app.

**Final status:** The code and baseline are committed and pushed. The research project is separately committed in its private repository. Release packaging passes, but the APK is unsigned; therefore, the release tag, GitHub release, APK asset, and direct download remain **BLOCKED / NOT CREATED** until the authorized signing key is securely available.

*Research metadata and repository states are point-in-time findings reviewed on 2026-09-24. Reverify upstream revisions and license terms before future reuse.*

*References 8 and 9 will resolve after this report is committed and pushed to the release branch.*

*The release source commit remains `05136cf7f4e1146d0380a287738aa76889c8b4e4`; this report will be added in a separate documentation commit.*

*No release signing credentials or private keys were requested in chat, displayed, or added to Git.*

*The local unsigned APK is a build artifact only and is not attached as a distribution download.*

*The release branch retains the upstream runtime download origin by design; only the app update manifest was redirected to the target repository.*

*The actual release-source commit message is `release: prepare U&U v1.0.9`, which differs from the suggested template text.*

## References

[1]: https://github.com/usmanroba/Automation-AI "Automation-AI production repository"
[2]: https://github.com/techjarves/Mobile-Harness "Mobile Harness upstream repository"
[3]: https://github.com/usmanroba/Automation-AI/tree/manus/automation-ai-v1.0.9 "U&U v1.0.9 release branch"
[4]: https://github.com/usmanroba/Automation-AI/tree/manus/automation-ai-baseline "Mobile Harness engineering baseline branch"
[5]: https://github.com/usmanroba/researches "Private autonomous AI research repository"
[6]: https://github.com/techjarves/Mobile-Harness/releases/tag/v1.0.4 "Published upstream v1.0.4 release used for read-only signer comparison"
[7]: https://github.com/techjarves/Mobile-Harness/releases/tag/runtime-2026.09.4 "Upstream shared runtime bundle release"
[8]: https://github.com/usmanroba/Automation-AI/blob/manus/automation-ai-v1.0.9/docs/V1_0_9_VALIDATION.md "U&U v1.0.9 build, test, and signing validation record"
[9]: https://github.com/usmanroba/Automation-AI/blob/manus/automation-ai-baseline/docs/MOBILE_HARNESS_BASELINE.md "Verified Mobile Harness source baseline"
[10]: https://github.com/usmanroba/researches/blob/main/project-index.md "Research project, license, and revision index"
