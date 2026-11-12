# Personal Android hardening baseline / 安卓个人加固基线

Date / 日期：2026-11-12 (Asia/Shanghai).
Maintenance branch / 维护分支：`kairyo/ClashMetaForAndroid:ai/security-hardening`.

## Source and package / 源码与安装包

- Upstream app: `f454fa8c569ea2a2396f2fe453ac8263832001d8` (v2.11.37 tag).
- Pinned mihomo: `0455d0d28c4526937380505153e65a843d8612e8`.
- Personal package: `io.kairyo.cmfa.hardened`, ARM64, `2.11.37-hardened.1`
  (`versionCode=21103701`). The official app's package is not replaced.
- Installed APK SHA-256:
  `569af4315f2b70420e0d2bf66a142bacc642be9cba2951e27ab73ce0536676cb`.
- Signing certificate SHA-256 (public fingerprint only / 仅公开指纹):
  `fd1f6b633bb227e8a559225abb934936814aa9cefd0d7beb1d3bd21707a78e32`.

The installation package and private signing material are retained locally, not
published in this branch. Publishing source does not create a binary release.
安装包与私有签名材料仅本机保留；本次只上传源码分支，不发布 APK Release。

## Changes / 修改内容

| Area / 范围 | Result / 结果 |
| --- | --- |
| Go dependencies / Go 依赖 | brotli 1.3.0; compress 1.20.4; DHCP `20261014062924-c6bb46ad3f3a`; x/crypto 0.59.0; x/net 0.62.0; x/sys 0.50.0; x/text 0.44.0; related dependency locks updated / 同步更新关联版本锁定。 |
| External control / 外部控制 | Start, stop and toggle require visible confirmation, including shortcuts / 启动、停止、切换及快捷方式均须确认。 |
| Backup / 备份 | Disable system backup and exclude all app data from cloud backup and device transfer / 禁止系统备份，排除云备份和设备迁移中的应用数据。 |
| Android TLS | Trust system CAs only; this does not define every Go TLS call's trust policy / 只信任系统 CA，不将此推论扩大到所有 Go TLS 调用。 |
| Signing logs / 签名日志 | Remove workflow steps printing signing properties / 删除输出签名配置的步骤。 |
| Personal app / 个人应用 | Independent package/name/version and ARM64 build / 独立包名、名称、版本及 ARM64 构建。 |
| Maintenance / 维护 | Add project instructions; remove push-triggered builds/pre-releases and external-dispatch dependency updates; adapt PR/manual validation to the personal ARM64 package / 补充项目规则，移除 push 构建与预发布、外部事件触发依赖更新；将 PR/手动验证适配为个人 ARM64 包。 |

Domestic DIRECT rules, per-app routing and configurable Allow Bypass are preserved
at the user's explicit request. They are not silently removed as security fixes.
按用户明确要求，保留国内直连、按应用分流及“允许应用绕过”；默认绕过设置沿用上游。

## Completed validation / 已完成验证

- `app:assembleMetaRelease` succeeded, including release vital lint.
  发布构建及其 vital lint 检查通过。
- Brotli, compress/fse, compress/huff0, x/net/http2 and dnsmessage package tests
  passed. 五组相关依赖测试通过。
- The delivered APK uses Go 1.26.11. OSV queries for 122 versions extracted from
  the actual APK returned no matches on the audit date. 实际 APK 提取的 122 项 Go
  版本查询在该日期为 0 命中，不能解释为“没有任何漏洞”。
- Three additional matches in the 177-entry source-module graph (ebpf, goldmark,
  grpc) were absent from the Android package graph and delivered APK module list.
  三组额外源码依赖告警未进入安卓包图及成品依赖列表。
- Signature schemes v1/v2/v3 verified; merged manifest and decoded backup/TLS
  resources matched the intended changes. 签名及 APK 内配置已核验。
- Installed on an Android 14 ARM64 device; the installed APK hash matched the
  delivered file. 在 Android 14 ARM64 真机安装，安装后的 APK 摘要一致。
- External START/STOP/TOGGLE each displayed confirmation. With the VPN initially
  stopped, cancelling each request left it stopped. 三种请求均显示确认；从未启动状态
  取消请求后，VPN 保持未启动。
- The user subsequently reported that the app works. 用户随后反馈可以使用；这与
  独立的网络安全验收是不同证据。

The app source used for the validated APK is unchanged by the later repository
publication step. That step adds maintenance documentation, workflow trigger
restrictions and validation-build wiring; no rebuilt binary is claimed for those
documentation/CI-only edits. Workflow YAML and intended events/paths were checked
locally; no GitHub Actions run was dispatched for this branch-only publication.
后续分支上传准备仅补充维护文档与 CI 配置，应用源码保持已验证版本；不虚称重新构建。
工作流 YAML、触发事件及路径已本地检查；本次只上传分支，没有主动运行 GitHub Actions。

## Remaining limits / 未验证项

- Version matching is not exploit reproduction, a complete JVM/native-library
  audit or a guarantee against unknown vulnerabilities / 版本查询不等于攻击复现、
  完整 JVM/原生库审计或未知漏洞保证。
- Google ML Kit remains in the QR scanner; no zero-telemetry claim / 扫码仍包含
  Google ML Kit，不宣称零遥测。
- No independent packet capture, DNS/IPv6 leak test, domestic routing test,
  network-switching/failover test or long-duration validation was completed.
  未独立完成抓包、DNS/IPv6 泄漏、国内分流、切网恢复及长时间运行测试。
- Cancelling an external stop request against an already running VPN was not
  tested end-to-end / 未对运行中的 VPN 做外部停止请求的完整攻击验收。
- Subscriptions and proxy operators remain separate trust decisions. No private
  subscription, node credentials or device screenshots are published here.
  订阅与节点提供方仍需单独判断信任，本分支不含私人订阅、节点凭据或设备截图。

Future work continues from this maintenance baseline, with scoped changes,
current evidence and explicit publication authorization. Retain the local signing
key for compatible app updates; do not replace it with a CI debug key.
后续以本分支为基线推进，按变更风险验收，并在授权范围内发布；妥善保留本机私钥，
不能用 CI 调试密钥替代已安装应用的更新签名。
