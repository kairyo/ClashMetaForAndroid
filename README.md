## Clash Meta for Android

### Personal hardened build / 个人加固版本

Maintenance repository: [kairyo/ClashMetaForAndroid](https://github.com/kairyo/ClashMetaForAndroid).
The user-selected maintenance baseline is `ai/security-hardening`; upstream `main`
is not the personal app release branch. See [AGENTS.md](AGENTS.md) for the workflow
and [the baseline validation record](docs/hardening-baseline-2026-11-12.md) for
tested behavior and remaining limits.

维护仓库为 `kairyo/ClashMetaForAndroid`，用户指定 `ai/security-hardening` 作为后续维护基线。
项目规则与本次验证记录见上述文件；不将上游 `main` 当作个人安装包的发布分支。

This local branch starts from upstream v2.11.37 (`f454fa8c`) with mihomo
`0455d0d`. It targets ARM64 Android devices and keeps rule-based DIRECT routing,
per-app routing, and the configurable Allow Bypass option. It is an independent
personal build, not an official MetaCubeX release or a guarantee of anonymity.

此本地分支基于上游 v2.11.37，面向 ARM64 安卓设备。保留国内网站直连、应用分流和
“允许应用绕过”开关；这是个人构建，不代表 MetaCubeX 官方发布，也不承诺匿名性。

- External start/stop/toggle requests require on-screen confirmation, including
  launcher shortcuts. 外部启停请求（包括桌面快捷方式）须经屏幕确认。
- System backup and device migration exclude app data; subscriptions must be
  imported again on a new installation. 禁止系统备份和设备迁移复制敏感配置，新安装需重新导入。
- Android TLS uses system CAs only. Android 网络请求不再额外信任用户安装的证书。
- The Android Go build module pins updated HTTP/2, crypto, DNS/text and compression
  dependencies in `core/src/foss/golang/go.mod`. 以此模块和 `go.sum` 为安卓构建依赖依据。
- CI no longer prints signing properties. CI 不再输出签名配置文件。

Build requirements: JDK 21, Android SDK 35, NDK 29.0.14206865, CMake 3.22.1,
and Go 1.26.11 with `.github/patch/disable_pidfd_on_android.patch` applied to
that isolated Go installation. Use `custom.application.id=io.kairyo.cmfa.hardened`
and `remove.suffix=true` in the ignored `local.properties`, then run
`./gradlew --no-daemon app:assembleMetaRelease`.

构建需要上述工具版本；仅修改专用 Go 工具目录，不修改系统工具链。使用独立包名
`io.kairyo.cmfa.hardened`，避免覆盖官方应用。若未提供发布签名，Gradle 会使用调试密钥，
该产物仅用于构建验证；正式安装包须另行使用个人私钥签名，并妥善保留私钥以便更新。

Known limits: the QR scanner still includes Google ML Kit; this branch does not
claim zero telemetry. Allow Bypass intentionally remains available. Imported
subscriptions and proxy operators remain separate trust decisions. Device VPN,
DNS/IPv6, failover and domestic DIRECT routing require runtime acceptance.

已知边界：扫码仍包含 Google ML Kit，不能宣称零遥测；绕过 VPN 的能力按用户需要保留。
订阅和节点提供方仍需单独判断信任。VPN、DNS/IPv6、故障恢复及国内直连须以实际测试验收。

A Graphical user interface of [Clash.Meta](https://github.com/MetaCubeX/Clash.Meta) for Android

### Feature

Feature of [Clash.Meta](https://github.com/MetaCubeX/Clash.Meta)

[<img src="https://fdroid.gitlab.io/artwork/badge/get-it-on.png"
     alt="Get it on F-Droid"
     height="80">](https://f-droid.org/packages/com.github.metacubex.clash.meta/)

### Requirement

- Android 5.0+ (minimum)
- Android 7.0+ (recommend)
- `armeabi-v7a` , `arm64-v8a`, `x86` or `x86_64` Architecture

### Build

1. Update submodules

   ```bash
   git submodule update --init --recursive
   ```

2. Install **OpenJDK 11**, **Android SDK**, **CMake** and **Golang**

3. Create `local.properties` in project root with

   ```properties
   sdk.dir=/path/to/android-sdk
   ```

4. (Optional) Custom app package name. Add the following configuration to `local.properties`.

   ```properties
   # config your ownn applicationId, or it will be 'com.github.metacubex.clash'
   custom.application.id=com.my.compile.clash
   # remove application id suffix, or the applicaion id will be 'com.github.metacubex.clash.alpha'
   remove.suffix=true

5. Create `signing.properties` in project root with

   ```properties
   keystore.path=/path/to/keystore/file
   keystore.password=<key store password>
   key.alias=<key alias>
   key.password=<key password>
   ```

6. Build

   ```bash
   ./gradlew app:assembleAlphaRelease
   ```

### Automation

APP package name is `com.github.metacubex.clash.meta`

- Toggle Clash.Meta service status
  - Send intent to activity `com.github.kr328.clash.ExternalControlActivity` with action `com.github.metacubex.clash.meta.action.TOGGLE_CLASH`
- Start Clash.Meta service
  - Send intent to activity `com.github.kr328.clash.ExternalControlActivity` with action `com.github.metacubex.clash.meta.action.START_CLASH`
- Stop Clash.Meta service
  - Send intent to activity `com.github.kr328.clash.ExternalControlActivity` with action `com.github.metacubex.clash.meta.action.STOP_CLASH`
- Import a profile
  - URL Scheme `clash://install-config?url=<encoded URI>` or `clashmeta://install-config?url=<encoded URI>`

### Contribution and Project Maintenance

#### Meta Kernel

- CMFA uses the kernel from `android-real` branch under `MetaCubeX/Clash.Meta`, which is a merge of the main `Alpha` branch and `android-open`.
  - If you want to contribute to the kernel, make PRs to `Alpha` branch of the Meta kernel repository.
  - If you want to contribute Android-specific patches to the kernel, make PRs to  `android-open` branch of the Meta kernel repository.

#### Maintenance

- When `MetaCubeX/Clash.Meta` kernel is updated to a new version, the `Update Dependencies` actions in this repo will be triggered automatically.
  - It will pull the new version of the meta kernel, update all the golang dependencies, and create a PR without manual intervention.
  - If there is any compile error in PR, you need to fix it before merging. Alternatively, you may merge the PR directly.
- Manually triggering `Build Pre-Release` actions will compile and publish a `PreRelease` version.
- Manually triggering `Build Release` actions will compile, tag and publish a `Release` version.
  - You must fill the blank `Release Tag` with the tag you want to release in the format of `v1.2.3`.
  - `versionName` and `versionCode` in `build.gradle.kts` will be automatically bumped to the tag you filled above.
