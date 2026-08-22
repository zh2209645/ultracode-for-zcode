# 维护任务清单

> MVP 阶段任务已于 2026-08-18 全部完成，插件效果评估通过；v0.1.5 为当前发布版本。
> 原始实施清单的逐项结果与证据保存在 `docs/08-TEST-RECORD.md` 与 `docs/09-FINAL-REPORT.md`，本文件不再重复，只跟踪维护期待办与例行事项。

## 当前状态（2026-08-23）

- 发布版本 v0.1.5（2026-08-19，GPG 签名 tag），marketplace ref 已固定。
- 强制回归 9 场景 + 3 探针全部通过（`docs/08` §14）。
- ZCode 3.7.7 与 3.8.1 核心链路复验通过（`docs/08` §11、§15）。
- WSL2 (Ubuntu) 核心复验通过，安装缓存与签名 tag 字节一致（`docs/08` §16）。

## 待办

- [ ] 评估 `worker-fast` 切换到 `glm-5-turbo`：先在本机新会话确认确切的 model ID 拼写与 `thoughtLevel` 取值，再改 frontmatter 并逐一实测（见 `plugins/zcode-dynamic-workflow/MODEL-MAPPING.md` Option A）。
- [ ] ZCode 发布新版本时复验核心链路：插件加载、`/ultracode` 挂载、agent 注册、Explore/worker 实际派发，并追加 `docs/08` 记录。
- [ ] 在更多平台（如 macOS）补充核心复验记录（现有 Windows 与 WSL2 证据）。
- [ ] 收集一段时间的实际使用反馈后，评估是否需要调整 `SKILL.md` 的请求定级与路由阈值。

## 例行发布检查单（每次发版勾选）

- [ ] 行为有变更时重跑强制回归 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14/F-15，并追加 `docs/08` 记录
- [ ] 同步三处版本号：`plugin.json`、`marketplace.json`（`version` + `ref`）、根 `README.md`
- [ ] 重新生成 `SHA256SUMS.txt`
- [ ] `git tag -s` 签名发布并推送，marketplace `ref` 固定到新 tag
- [ ] 新会话安装/更新验证 + 版本一致性检查
- [ ] 确认未引入 Hook / MCP / 额外 runtime / 持久状态

## 历史里程碑

- 2026-08-18 — 0.1.0 MVP 完成，F-01–F-12 全过（`docs/08` §2）
- 2026-08-19 — 0.1.2 安全整改；0.1.3 经 GitHub 应用内更新安装并完成 8 场景回归（`docs/08` §11）
- 2026-08-19 — 0.1.4 签名发布，收口 S-12（版本同步、ref 固定、签名 tag，`docs/08` §12）
- 2026-08-19 — 0.1.5 发布：请求定级、上下文卫生、orchestrator 模式；9 场景 + 3 探针回归通过（`docs/08` §14）
- 2026-08-19 — ZCode 3.8.1 升级后核心复验通过（`docs/08` §15）
- 2026-08-23 — WSL2 核心复验通过，首个非 Windows 主机记录（`docs/08` §16）
