# 06 — 可直接交给本地 LLM 的主提示

下面内容可以整段复制给负责迁移和重构的本地 LLM。先把路径和模型 ID 占位符替换为实际值。

---

## 任务

你正在实现一个 ZCode 原生插件，工作名为 `zcode-dynamic-workflow`。源思想来自 oh-my-claudecode 的 ultrawork 和 agent tier 路由，但禁止全量迁移 OMC。

目标是交付第一版 Dynamic Workflow 能力：

- 动态规划完全由 ZCode 主 Agent决定；
- 插件只教主 Agent如何根据任务难度、风险和独立性委派；
- 使用 ZCode 内置 Explore，以及 fast / standard / deep 三个通用 subagent；
- 主 Agent保留总任务图、并发波次、结果整合、升级和最终验证；
- 不实现任何重型 harness。

## 工作目录

- 迁移资料：`<MVP_KIT_PATH>`
- OMC 源仓库或参考目录：`<OMC_SOURCE_PATH>`
- 目标实现目录：`<TARGET_PATH>`

## 模型映射

- Fast model ID：`<FAST_MODEL_ID>`
- Standard model ID：`<STANDARD_MODEL_ID>`
- Deep model ID：`<DEEP_MODEL_ID>`

若三个层级使用同一模型，请分别设置该模型支持的低、中/高、最高 thought level。不要猜 model id；优先从本机 ZCode 已连接模型配置中读取或由用户提供。

## 必读资料

按顺序阅读：

1. `<MVP_KIT_PATH>/AGENTS.md`
2. `<MVP_KIT_PATH>/ZCODE_DYNAMIC_WORKFLOW_MVP.md`
3. `<MVP_KIT_PATH>/docs/03-DELEGATION-POLICY.md`
4. `<MVP_KIT_PATH>/docs/04-MIGRATION-PLAN.md`
5. `<MVP_KIT_PATH>/docs/05-ACCEPTANCE-TESTS.md`
6. `<MVP_KIT_PATH>/references/zcode/ZCODE_CAPABILITY_NOTES.md`
7. `<MVP_KIT_PATH>/references/omc/ultrawork.SKILL.md`
8. `<MVP_KIT_PATH>/references/omc/agent-tiers.md`

## 强制范围

只允许实现：

```text
marketplace.json
plugins/zcode-dynamic-workflow/.zcode-plugin/plugin.json
plugins/zcode-dynamic-workflow/commands/ultracode.md
plugins/zcode-dynamic-workflow/skills/dynamic-workflow/SKILL.md
plugins/zcode-dynamic-workflow/agents/worker-fast.md
plugins/zcode-dynamic-workflow/agents/worker-standard.md
plugins/zcode-dynamic-workflow/agents/worker-deep.md
必要的 README、MODEL-MAPPING、LICENSE 和测试记录
```

禁止实现：

- Hook；
- MCP；
- Node、TypeScript、Python 或 shell runtime；
- Team、swarm、mailbox、heartbeat、shared queue；
- Ralph 或持久循环；
- 状态目录或数据库；
- HUD、通知、token 统计；
- worktree 编排；
- 多 CLI provider；
- nested subagent；
- 固定 workflow DAG；
- 自动模型发现框架。

## 设计要求

1. 主 Agent是唯一 orchestrator。
2. Skill 只定义决策政策，不替主 Agent写死计划。
3. 使用五维评分：范围、模糊度、耦合与推理、风险、验证成本。
4. 默认路由：
   - 0–2：主 Agent直接；
   - 3–4：fast；
   - 5–7：standard；
   - 8–10：deep。
5. 只读搜索使用内置 Explore。
6. 安全、认证、授权、数据、公开 API、架构和复杂重构至少使用 deep 分析或 deep 复核。
7. 默认最大并发 10。
8. 同文件写任务和有依赖任务不得错误并行。
9. fast 失败升级 standard；standard 失败或风险扩大升级 deep；deep 失败交回主 Agent重规划。
10. 同一任务最多自动升级一次。
11. worker 不扩大范围、不再委派、不宣布总任务完成。
12. worker 返回 status、summary、files/evidence、verification、risks/blockers。
13. 主 Agent必须基于实际证据做最终验证。
14. 不使用 Claude Code 专用 `Task(...)` 语法；依赖 ZCode 原生 Agent 工具。
15. Prompt 保持短而明确，不复制 OMC 的长角色体系。

## 实施步骤

1. 检查现有骨架和 Git 状态。
2. 建立或修正本地 marketplace 和最小 plugin manifest。
3. 替换三个真实模型 ID 和合法 thought level。
4. 完成 `dynamic-workflow` Skill。
5. 完成三个 worker 的精简 system prompt。
6. 完成 `/ultracode` 命令。
7. 验证 JSON 和 frontmatter。
8. 在 ZCode 安装插件并新开 session。
9. 执行 `docs/05-ACCEPTANCE-TESTS.md`，至少完成 8 个功能场景，关键场景 F-01、F-02、F-05、F-07、F-09、F-12 必须通过。
10. 根据行为结果优先调整 description 和 Skill，不增加 runtime。
11. 更新许可证、来源说明和测试记录。
12. 生成最终报告。

## 代码与文档原则

- 最小变更；
- 不新增依赖；
- 优先删除无关 OMC 组件，而不是做兼容包装；
- 不保留 Claude 专用工具名和路径；
- 不发明未经 ZCode 文档确认的 frontmatter 字段；
- `thoughtLevel` 使用 camelCase；
- Plugin、Skill、Agent 目录遵循 ZCode 规范；
- 模型占位符不得留在最终版本；
- 对无法实测的行为明确标记，不伪造通过。

## 完成条件

只有满足以下条件才能宣布完成：

- 插件可加载；
- Skill、Command、三个 Agent 均可见；
- 三个 Agent 均可成功调用；
- 路由和波次测试达到文档要求；
- 无 Hook、MCP、runtime、持久状态；
- 无 nested delegation；
- 模型映射和安装步骤写清；
- MIT 许可证与上游致谢完整；
- 最终报告列出所有修改、测试证据、失败项和剩余风险。

## 最终输出格式

```text
## Summary

## Files Added / Changed / Removed

## Model Mapping

## Static Validation

## Functional Scenarios
| ID | Result | Route | Evidence |

## Scope Guard
- Hooks:
- MCP:
- Runtime:
- Persistent state:
- Nested delegation:

## Remaining Risks

## Manual Steps
```

直接开始执行。不要把任务扩展为 OMC 全量移植。
