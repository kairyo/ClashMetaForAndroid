# CMFA Hardened — repository instructions / 仓库规则

Adapted from the agents-md AGENTS template at v0.2.0 / `b706de1`.
采用固定规范：[agents-md v0.2.0](https://github.com/maweis1981/agents-md/blob/b706de13e23d8d7435af0d96c3478e3352901fcd/STANDARDS.zh-CN.md)
及用户选择性采用的 [agent-work-standards 0.1.5-draft](https://github.com/kairyo/agent-work-standards/blob/f6b63090911d4c16a82bc01895cbc72e0506f673/DRAFT.zh-CN.md)。
不要自动跟随这些规范的 live main。当前用户指令和平台权限优先。

## Ownership and branches / 归属与分支

- Only publish this project's work to `kairyo/ClashMetaForAndroid` unless the user
  explicitly authorizes another destination. `MetaCubeX/ClashMetaForAndroid` is
  the read-only upstream reference, not a push target.
- 仅向自己的 Fork 提交项目工作；未经用户另行授权，不向上游推送、提 Issue 或 PR。
- On 2026-11-12 the user explicitly chose `ai/security-hardening` as the ongoing
  maintenance baseline. This named baseline is the user-authorized exception to
  the usual short-lived branch rule. Do not merge it into `main` or change the
  default branch automatically.
- 用户指定 `ai/security-hardening` 为后续维护基线。后续功能从该基线创建短生命周期的
  `ai/<feature>` 分支；需要合入时，通过自己仓库内的单功能 PR，经授权后 squash 合并。
- Commits follow Conventional Commits and group one stable logical change.
  Commit/push/PR/release authorization is determined by the current user request;
  permission to upload this baseline is not perpetual publication authorization.
- GitHub Issue/PR titles, descriptions and status comments are bilingual Chinese
  and English. Do not send project communications without applicable authorization.

## Product and private data / 产品与敏感资料

- Keep domestic DIRECT rules, per-app routing and configurable Allow Bypass.
  用户已明确需要国内直连和绕过能力，不能将它们误作漏洞删除。
- Keep package identity `io.kairyo.cmfa.hardened` for personal release builds.
  Do not overwrite the official app or user configuration without authorization.
- Never commit/upload signing keys, signing passwords, `local.properties`, APKs,
  subscriptions, node credentials, private screenshots or raw device data.
  Signing material remains local and must be retained for compatible updates.
- Preserve upstream license/copyright notices. Do not present this fork as an
  official MetaCubeX release or claim zero vulnerabilities/zero telemetry.

## Build and evidence / 构建与验证

- Stack: Kotlin/Android, JNI/CMake, Go/mihomo. The tested baseline uses JDK 21,
  SDK 35, NDK 29.0.14206865, CMake 3.22.1 and Go 1.26.11 with the repository's
  Android pidfd patch applied to an isolated Go installation; see README.
- The Android entry module is `core/src/foss/golang/go.mod`, which replaces
  `cfa` and the pinned mihomo submodule locally. Check the actual APK build info;
  do not infer Android dependency versions from another module's go.mod.
- Release build: `./gradlew --no-daemon app:assembleMetaRelease`. This includes
  release vital lint. Without release signing configuration, upstream Gradle
  falls back to a debug key; re-sign with the retained personal key before delivery.
- Match tests to changes. The baseline dependency checks ran from
  `core/src/foss/golang`:

  ```sh
  go test -timeout 120s github.com/andybalholm/brotli github.com/klauspost/compress/fse github.com/klauspost/compress/huff0 golang.org/x/net/http2 golang.org/x/net/dns/dnsmessage
  ```

- Verify the final APK's signature, package/ABI, merged manifest, backup/TLS
  resources and relevant dependency versions. A clean OSV version query is not
  exploit testing or a complete Android/JVM/native-library audit.
- UI changes require actual screen and interaction checks. Distinguish static,
  build, device and user-reported results; do not claim DNS/IPv6 or failover tests
  that were not performed. Stop device automation if the user is editing secrets.

## CI and finishing work / CI 与收尾

- CI must not publish releases on push. Debug validation may run on PR/manual
  events; dependency updates and inherited release workflows are manual only.
- Inherited release workflows still contain upstream version/tag/signing behavior.
  Do not run them for personal releases before reviewing and adapting them to the
  authorized maintenance branch and signing process. No automatic main pushes.
- Before pushing: inspect staged paths and diff, check for credentials, verify
  the intended owner/branch and record relevant completed checks and limits.
- Do not overwrite concurrent work, force-push or bypass repository protections.
  Cap same-failure retries at three with a new hypothesis each time.
- Keep temporary build/download/test files in a task directory; remove disposable
  files after retaining minimal evidence. Preserve source, current APK and signing
  material for their documented maintenance/update needs.
