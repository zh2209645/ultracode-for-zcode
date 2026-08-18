# 本地 LLM 项目指令：ZCode Dynamic Workflow MVP

## 任务使命

在不移植 OMC 重型运行时的前提下，实现一个 ZCode 原生插件，使主 Agent 能够：

1. 自行理解用户目标并动态拆分任务；
2. 判断哪些工作应直接完成、哪些应委派；
3. 按难度选择 `worker-fast`、`worker-standard` 或 `worker-deep`；
4. 使用 ZCode 内置 `Explore` 进行只读代码库探索；
5. 使用高性能模型驱动的只读 `worker-review` 进行高风险独立复核、裁决和验收证据检查，一般探索与普通分析不使用该 Agent；
6. 将独立任务并行，将有依赖任务按波次执行；
7. 汇总结果、在必要时升级任务层级，并执行最低限度验证。

## 必读文件

开始修改前，按顺序阅读：

1. `ZCODE_DYNAMIC_WORKFLOW_MVP.md`
2. `docs/03-DELEGATION-POLICY.md`
3. `docs/04-MIGRATION-PLAN.md`
4. `docs/05-ACCEPTANCE-TESTS.md`
5. `plugins/zcode-dynamic-workflow/MODEL-MAPPING.md`
6. `references/zcode/ZCODE_CAPABILITY_NOTES.md`
7. `references/omc/ultrawork.SKILL.md`
8. `references/omc/agent-tiers.md`

## 强制范围

首版只实现：

- `.zcode-plugin/plugin.json`
- `skills/dynamic-workflow/SKILL.md`
- `commands/ultracode.md`
- `agents/worker-fast.md`
- `agents/worker-standard.md`
- `agents/worker-deep.md`
- `agents/worker-review.md`
- 必要的说明文档和本地 marketplace manifest

禁止加入：

- Team、swarm、mailbox、heartbeat、shared queue；
- Ralph、持久循环、恢复状态；
- Hook；
- MCP server；
- Node/TypeScript/Python 运行时；
- HUD、token 统计、通知；
- worktree 自动管理；
- 多供应商 CLI 调度；
- 自动模型发现或复杂配置生成器；
- subagent 再派生 subagent 的设计。

## 架构原则

- **主 Agent 是唯一 orchestrator。**
- **主 Agent 保留任务图、依赖关系、结果整合和最终结论。**
- **subagent 只完成被分配的原子任务，不重新定义总目标。**
- **不为简单任务委派。**
- **不把本来有依赖的任务伪装成并行任务。**
- **优先使用 ZCode 原生能力，不发明未在文档中出现的工具调用语法。**
- **提示词保持短而明确，不复制 OMC 的全部长提示。**
- **没有证据时不声称验证通过。**

## 实施策略

1. 先检查当前骨架是否能被 ZCode 识别。
2. 找到本机真实模型 ID，配置四个 Agent。
3. 保持一个 Skill、三个可写 worker 和一个只读 reviewer 的最小结构。
4. 通过 Agent 的 `description` 提高自动选择准确率。
5. 让 Skill 定义动态拆解和路由规则，而不是写死固定流水线。
6. 用场景测试验证委派行为，而不是只检查文件能加载。
7. 发现问题时先调整 Skill 和 Agent 描述，除非确有必要，不新增代码层。

## 许可证

OMC 使用 MIT License。若直接复制或实质性改写上游文本，必须在分发物中保留上游版权和许可声明。首版建议“提炼思想、重写文本”，同时保留 `references/omc/LICENSE` 和项目致谢。

## 完成报告

最终报告必须包含：

- 修改和新增的文件；
- 三个模型层级的实际映射；
- 每个验收场景的结果；
- 未通过项及原因；
- 仍需人工完成的配置；
- 是否满足“无 Hook、无 MCP、无额外 runtime、无持久状态”。
