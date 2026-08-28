# 维护任务清单

> MVP 阶段任务已于 2026-08-18 全部完成，插件效果评估通过；v0.2.0 为当前发布版本。
> 原始实施清单的逐项结果与证据保存在 `docs/08-TEST-RECORD.md` 与 `docs/09-FINAL-REPORT.md`，本文件不再重复，只跟踪维护期待办与例行事项。

## 当前状态（2026-08-27）

- 发布版本 v0.2.0（2026-08-27，GPG 签名 tag），marketplace ref 已固定。
- 强制回归 9 场景 + 3 探针全部通过（`docs/08` §14）。
- ZCode 3.7.7 与 3.8.1 核心链路复验通过（`docs/08` §11、§15）。
- WSL2 (Ubuntu) 核心复验通过，安装缓存与签名 tag 字节一致（`docs/08` §16）。
- ZCode 3.9.1 核心链路复验通过，四类 worker 全部活体派发（`docs/08` §17；官方 changelog 确认 3.8.1→3.9.1 未触及插件依赖面）。
- 2026-08-27：worker-fast/worker-standard 模型映射切换为 GLM-5.3-Flash（thoughtLevel high/max，字段名以官方文档为准），静态验证通过；运行时验证与强制回归待新版本发布后的新会话（`docs/08` §18）。
- 2026-08-27：新增第五个 agent `explore-flash`（中低难度只读探索，glm-5.3-flash + thoughtLevel low——同日由 high 调整，description 明示局限），SKILL/docs03/docs05/AGENTS/README/marketplace 同步，产品边界改为五 agent 口径；静态验证通过，运行时验证与强制回归待发布后新会话（`docs/08` §19）。

## 待办

- [x] 评估 `worker-fast` 切换到 `glm-5-turbo` —— 已被取代：2026-08-27 随 ZCode 3.9.2 直接将 worker-fast/worker-standard 切换为 `glm-5.3-flash`（`docs/08` §18）。
- [x] 新版本发布后的新会话实测 GLM-5.3-Flash 映射与 explore-flash：确认 qualified model ID 解析（回退顺序见 `docs/04` §4）、`thoughtLevel` low/high/max 生效、五 agent 注册与 explore-flash 只读派发/拒写/升级路径，随后按发布检查单重跑强制回归 9 场景 + 3 探针并追加 `docs/08` 记录（`docs/08` §18、§19）—— 已于 2026-08-27 ZCode 3.9.2 新会话完成（`docs/08` §21）；但其中「explore-flash partial→Explore 升级路径」子项为策略已验证/运行时未触发（触发条件未自然出现），如需运行时证据待后续真实 partial 场景补充。
- [ ] ZCode 发布新版本时复验核心链路：插件加载、`/ultracode` 挂载、agent 注册、Explore/worker 实际派发，并追加 `docs/08` 记录。——实例记录：3.9.2 已复验（2026-08-27，`docs/08` §21）；3.10.1 已复验（2026-08-29，`docs/08` §22）。
- [ ] 在更多平台（如 macOS）补充核心复验记录（现有 Windows 与 WSL2 证据）。
- [ ] 收集一段时间的实际使用反馈后，评估是否需要调整 `SKILL.md` 的请求定级与路由阈值。已记数据点：explore-flash（low 档）在 2026-08-27 会话完成 32 文件全仓穷尽盘点且按文件枚举完整（自报计数 32 vs 实际 33，差 1），提示低成本档能力上限高于阈值假设，供路由阈值评估参考（`docs/08` §21）。

## 例行发布检查单（每次发版勾选）

- [ ] 行为有变更时重跑强制回归 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14/F-15，并追加 `docs/08` 记录
- [ ] 同步四处版本号：`plugin.json`、`marketplace.json`（`version` + `ref`）、根 `README.md`、SKILL.md `metadata.version`
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
- 2026-08-25 — ZCode 3.9.1 升级后核心复验通过，四类 worker 活体派发（`docs/08` §17）
- 2026-08-27 — worker-fast/worker-standard 切换 GLM-5.3-Flash（ZCode 3.9.2），静态验证通过，运行时验证待发布后新会话（`docs/08` §18）
- 2026-08-27 — 新增第五个 agent explore-flash（低成本只读探索档），产品边界改五 agent 口径，静态验证通过（`docs/08` §19）
- 2026-08-27 — v0.2.0 发布：explore-flash 五 agent、GLM-5.3-Flash 混合映射、难度分流只读路由；两轮发布前审核 GO，GPG 签名 tag（cd51a00）推送完成（`docs/08` §20）
