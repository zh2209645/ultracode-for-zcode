# 项目指令：zcode-dynamic-workflow（维护期）

## 项目状态

`zcode-dynamic-workflow` 是一个纯声明式 ZCode 插件（JSON + Markdown，无任何可执行代码），为 ZCode 主 Agent 提供动态任务拆分与委派策略。MVP 已于 2026-08-18 完成，插件效果评估全部通过。当前发布版本 **v0.2.0**（2026-08-27，GPG 签名 tag），marketplace ref 已固定到该 tag。v0.2.0 已于 2026-08-27 在 ZCode 3.9.2 新会话完成发布后全量验证（安装链路、五 agent 活体派发、9 场景 + 3 探针，`docs/08` §21）。宿主 3.9.2 → 3.10.1 更新后的核心链路复验已于 2026-08-29 完成（`docs/08` §22）。项目处于维护期。

## 仓库结构

```text
marketplace.json                  本地 marketplace manifest（ref 固定到签名 tag）
SHA256SUMS.txt                    发布物校验和
plugins/zcode-dynamic-workflow/   插件发布目录
├── .zcode-plugin/plugin.json     插件 manifest（版本号在此维护）
├── commands/ultracode.md         /ultracode 编排模式命令
├── skills/dynamic-workflow/SKILL.md  动态拆解与路由规则（行为核心）
├── agents/explore-flash.md       中低难度只读探索（glm-5.3-flash，thoughtLevel: low）
├── agents/worker-fast.md         低难度写任务（glm-5.3-flash，thoughtLevel: high）
├── agents/worker-standard.md     常规写任务（glm-5.3-flash，thoughtLevel: max）
├── agents/worker-deep.md         高难度写任务（thoughtLevel: max）
├── agents/worker-review.md       只读高风险复核（thoughtLevel: max）
├── MODEL-MAPPING.md              模型映射与更换流程
├── README.md / LICENSE
docs/01–09                        设计、政策、验收、测试记录、最终报告
references/                       上游 OMC 与 ZCode 能力参考资料（只读）
TASKS.md                          维护任务清单（待办与例行检查）
```

## 文档地图

维护时按需查阅，无需按固定顺序通读：

- 委派规则与任务契约：`plugins/zcode-dynamic-workflow/skills/dynamic-workflow/SKILL.md`、`docs/03-DELEGATION-POLICY.md`
- 模型映射与更换步骤：`plugins/zcode-dynamic-workflow/MODEL-MAPPING.md`
- 验收场景定义：`docs/05-ACCEPTANCE-TESTS.md`
- 各版本测试证据与平台复验记录：`docs/08-TEST-RECORD.md`
- 设计背景与架构：`ZCODE_DYNAMIC_WORKFLOW_MVP.md`、`docs/02-MVP-ARCHITECTURE.md`
- 上游参考（只读，不随发布分发）：`references/omc/`、`references/zcode/`

## 产品边界（长期不变式）

以下限制是产品决策，不是待办事项。任何突破都必须先单独讨论、记录理由并重跑回归：

- 不引入 Hook、MCP server、Node/TypeScript/Python 运行时或持久状态；
- 不引入 Team/swarm/mailbox/heartbeat、Ralph 循环、HUD、自动 worktree 管理、多供应商 CLI 调度；
- 不允许 subagent 再派生 subagent；
- 发布物只包含 plugin manifest、一个 Skill、一个 command、五个 agent 定义和说明文档（2026-08-27 经用户决策新增 explore-flash 低成本只读探索档，见 docs/08 §19）；
- 五个 agent 均设置 `injectAgentsMd: false`；写 worker 工具白名单为 Read/Edit/Write/Glob/Grep，reviewer 与 explore-flash 为 Read/Glob/Grep，均无 shell、网络和 MCP 工具。

## 架构原则

- **主 Agent 是唯一 orchestrator，保留任务图、依赖关系、结果整合和最终结论。**
- **subagent 只完成被分配的原子任务，不重新定义总目标。**
- **不为简单任务委派；不把有依赖的任务伪装成并行任务。**
- **中低难度探索走 `explore-flash`（低成本），高难度或广度探索走内置 `Explore`，高风险独立复核走 `worker-review`，一般分析三者都不用。**
- **只使用 ZCode 已文档化的原生能力，不发明新的工具调用语法。**
- **提示词保持短而明确。**
- **没有证据时不声称验证通过。**

## 修改约定

1. 行为问题优先通过调整 `SKILL.md` 和 agent `description` 解决，不新增代码层或新文件类型。
2. 更换模型时按 `MODEL-MAPPING.md` 的流程修改五个 agent frontmatter，并在新会话逐一实测确认生效。
3. 改动政策或模型后必须重跑回归（见下节），证据追加到 `docs/08-TEST-RECORD.md` 对应小节。
4. 同步维护 `TASKS.md`：新发现的改进项和例行检查结果记录在那里。

## 验证与回归

- 强制回归场景：F-01 / F-02 / F-05 / F-07 / F-09 / F-12 / F-13 / F-14 / F-15（定义见 `docs/05`，最近一次全过：v0.2.0（ZCode 3.9.2），9 场景 + 3 探针（升级路径子项为策略级验证），`docs/08` §21）。
- 重跑触发条件：SKILL.md / command / agent 政策变更，模型映射变更，ZCode 主版本更新。
- 仅文档或发布元数据变更可不重跑行为回归，但需完成静态校验与新会话安装验证。
- 平台核心复验（加载、挂载、注册、实际派发）已有 Windows（3.7.7 / 3.8.1 / 3.9.1 / 3.9.2 / 3.10.1）与 WSL2 记录（`docs/08` §11、§15、§16、§17、§21、§22），新平台首次使用时照此格式补充。

## 发布流程

1. 同步四处版本号：`plugins/zcode-dynamic-workflow/.zcode-plugin/plugin.json`、`marketplace.json`（`version` 与 `ref`）、根 `README.md` 的 "Current release"、`plugins/zcode-dynamic-workflow/skills/dynamic-workflow/SKILL.md` 的 `metadata.version`。
2. 重新生成 `SHA256SUMS.txt`。
3. 推送 main 后打 GPG 签名 tag（`git tag -s vX.Y.Z`）并推送，marketplace `ref` 固定到新 tag。
4. 在新会话安装/更新，验证 Skill 加载、`/ultracode` 可调用、五类 agent 可实际派发。
5. 回滚策略见 `docs/04-MIGRATION-PLAN.md` §9。

## 许可证

本项目按 MIT License 分发。委派概念提炼自 oh-my-claudecode（MIT）；修改相关文本时保留 `NOTICE.md` 致谢与 `references/omc/LICENSE` 中的上游版权和许可声明。
