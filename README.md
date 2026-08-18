# ZCode Dynamic Workflow MVP 迁移包

> 面向本地 LLM 的离线实施资料。目标是从 **oh-my-claudecode（OMC）** 中提炼最有价值的“按任务难度分级委派”思想，在 ZCode 中实现一个轻量、原生、无额外运行时的 Dynamic Workflow MVP。

## 1. 一句话目标

让 **ZCode 主 Agent 自己规划任务图**，并根据每个子任务的难度、风险和独立性，选择：

- 主 Agent 直接完成；
- ZCode 内置 `Explore` 做只读探索；
- `worker-fast` 完成简单、低风险任务；
- `worker-standard` 完成常规实现；
- `worker-deep` 完成复杂推理、高风险改动或独立复核。

插件不替主 Agent 做规划，也不实现 Team runtime、任务队列、心跳、状态机、Ralph 循环或 HUD。

## 2. 为什么采用这个范围

当前高性能模型已经具备较强的任务拆解、上下文理解和长程执行能力。首版真正有价值的不是再造一套重型 harness，而是向主 Agent 提供一套稳定、明确、低摩擦的委派策略：

1. **什么时候不应该委派**；
2. **什么时候使用只读探索 Agent**；
3. **如何判断子任务难度**；
4. **如何选择快、标准、深度三个性能层级**；
5. **如何把独立任务并行，把有依赖任务分波次执行**；
6. **如何整合结果、升级失败任务并完成最低限度验证**。

## 3. 包内内容

```text
zcode-dynamic-workflow-mvp-kit/
├── README.md
├── AGENTS.md
├── TASKS.md
├── ZCODE_DYNAMIC_WORKFLOW_MVP.md
├── marketplace.json
├── docs/
│   ├── 01-PROJECT-BRIEF.md
│   ├── 02-MVP-ARCHITECTURE.md
│   ├── 03-DELEGATION-POLICY.md
│   ├── 04-MIGRATION-PLAN.md
│   ├── 05-ACCEPTANCE-TESTS.md
│   ├── 06-LOCAL-LLM-PROMPT.md
│   └── 07-SOURCE-MAP.md
├── plugins/
│   └── zcode-dynamic-workflow/
│       ├── .zcode-plugin/plugin.json
│       ├── commands/ultracode.md
│       ├── skills/dynamic-workflow/SKILL.md
│       ├── agents/
│       │   ├── worker-fast.md
│       │   ├── worker-standard.md
│       │   └── worker-deep.md
│       ├── MODEL-MAPPING.md
│       └── LICENSE
└── references/
    ├── omc/
    │   ├── ultrawork.SKILL.md
    │   ├── agent-tiers.md
    │   ├── OMC_PLUGIN_MANIFEST_SNAPSHOT.json
    │   └── LICENSE
    └── zcode/
        └── ZCODE_CAPABILITY_NOTES.md
```

## 4. 给本地 LLM 的推荐阅读顺序

1. `AGENTS.md`
2. `ZCODE_DYNAMIC_WORKFLOW_MVP.md`
3. `docs/03-DELEGATION-POLICY.md`
4. `docs/04-MIGRATION-PLAN.md`
5. `docs/05-ACCEPTANCE-TESTS.md`
6. `plugins/zcode-dynamic-workflow/` 下的参考骨架
7. `references/` 下的离线参考材料

也可以直接把 `docs/06-LOCAL-LLM-PROMPT.md` 整份交给本地 LLM。

## 5. 首版交付物

首版完成后应只有以下运行组件：

- 1 个 ZCode 插件 manifest；
- 1 个动态委派 Skill；
- 1 个显式 `/ultracode` 命令；
- 3 个不同性能层级的通用 worker；
- 0 个 Hook；
- 0 个 MCP；
- 0 个 Node/TypeScript runtime；
- 0 个持久状态文件。

## 6. 使用前必须完成的配置

参考骨架里的三个 Agent 使用占位模型 ID：

- `REPLACE_WITH_FAST_MODEL_ID`
- `REPLACE_WITH_STANDARD_MODEL_ID`
- `REPLACE_WITH_DEEP_MODEL_ID`

实施者必须根据本机 ZCode 已连接的模型进行替换。若只有一个高性能模型，可使用同一模型 ID，并分别配置 `low`、`high`、`max` 思考等级，从而形成三个性能层级。

详见 `plugins/zcode-dynamic-workflow/MODEL-MAPPING.md`。

## 7. 来源快照

- 文档编制日期：2026-08-18
- OMC manifest 观察版本：`4.15.10`
- OMC 许可证：MIT
- ZCode 能力依据：官方 Plugin、Subagents、Skill、Hooks、Agent 文档

来源、兼容边界和许可证要求见 `docs/07-SOURCE-MAP.md`。
