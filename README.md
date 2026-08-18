# ZCode Dynamic Workflow MVP 迁移包

> 面向本地 LLM 的离线实施资料。目标是从 **oh-my-claudecode（OMC）** 中提炼最有价值的“按任务难度分级委派”思想，在 ZCode 中实现一个轻量、原生、无额外运行时的 Dynamic Workflow MVP。

## 0. 当前状态 / Current Status（2026-08-19）

- 当前版本 0.1.4：签名 tag `v0.1.4` 已发布（ed25519 GPG `6B699BE4A10CE49F`，`git verify-tag` good signature），marketplace source 固定该 tag，三处版本一致——S-12 全部通过，无剩余门禁项。
- 0.1.3 回归实测：强制场景 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14 全部 PASS（8/8），静态 S-03/S-04/S-05/S-06/S-10 通过；插件已经 GitHub marketplace 在应用内更新安装（ZCode 3.7.7）。完整记录见 `docs/08-TEST-RECORD.md` §11–§12 与 `docs/09-FINAL-REPORT.md`。
- 建议（非门禁）：GitHub 账号添加签名公钥以显示 Verified 徽标；ZCode 刷新 marketplace 更新至 0.1.4 并新开会话复验。

> **English abstract**: This kit distills the task-difficulty-based delegation idea from
> oh-my-claudecode into a lightweight, native ZCode plugin. The primary Agent plans the task
> graph itself and routes each atomic subtask to direct work, built-in `Explore`, read-only
> `worker-review`, or one of three write-capable tiers (`worker-fast` / `worker-standard` /
> `worker-deep`), running independent tasks as parallel waves and dependent tasks in order.
> The plugin is declarative Markdown/JSON only — no hooks, no MCP, no extra runtime, no
> persistent state. v0.1.3 passed all eight mandatory regression scenarios in-app on ZCode
> 3.7.7 (2026-08-19); the docs-only v0.1.4 release pins the marketplace to the GPG-signed
> `v0.1.4` tag, closing S-12.

## 1. 一句话目标

让 **ZCode 主 Agent 自己规划任务图**，并根据每个子任务的难度、风险和独立性，选择：

- 主 Agent 直接完成；
- ZCode 内置 `Explore` 做只读探索；
- `worker-fast` 完成简单、低风险任务；
- `worker-standard` 完成常规实现；
- `worker-deep` 完成明确需要写文件的复杂或高风险改动；
- `worker-review` 使用高性能模型和只读工具完成高风险独立复核、裁决和验收证据检查；一般探索与普通分析不使用它。

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
├── .gitattributes
├── .gitignore
├── README.md
├── AGENTS.md
├── NOTICE.md
├── TASKS.md
├── ZCODE_DYNAMIC_WORKFLOW_MVP.md
├── SHA256SUMS.txt
├── marketplace.json
├── docs/
│   ├── 01-PROJECT-BRIEF.md
│   ├── 02-MVP-ARCHITECTURE.md
│   ├── 03-DELEGATION-POLICY.md
│   ├── 04-MIGRATION-PLAN.md
│   ├── 05-ACCEPTANCE-TESTS.md
│   ├── 06-LOCAL-LLM-PROMPT.md
│   ├── 07-SOURCE-MAP.md
│   ├── 08-TEST-RECORD.md
│   └── 09-FINAL-REPORT.md
├── plugins/
│   └── zcode-dynamic-workflow/
│       ├── .zcode-plugin/plugin.json
│       ├── commands/ultracode.md
│       ├── skills/dynamic-workflow/SKILL.md
│       ├── agents/
│       │   ├── worker-fast.md
│       │   ├── worker-standard.md
│       │   ├── worker-deep.md
│       │   └── worker-review.md
│       ├── README.md
│       ├── MODEL-MAPPING.md
│       └── LICENSE
└── references/
    ├── omc/
    │   ├── README.md
    │   ├── ultrawork.SKILL.md
    │   ├── agent-tiers.md
    │   ├── executor-notes.md
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
- 3 个不同性能层级的可写 worker；
- 1 个高性能只读 reviewer；
- 0 个 Hook；
- 0 个 MCP；
- 0 个 Node/TypeScript runtime；
- 0 个持久状态文件。

## 6. 模型映射（已完成）

三个写 worker 和一个只读 reviewer 已配置为本机真实模型 ID（更新日期 2026-08-19）：

- `worker-fast` → `builtin:zai-coding-plan/GLM-5.3`，`thoughtLevel: low`
- `worker-standard` → `builtin:zai-coding-plan/GLM-5.3`，`thoughtLevel: high`
- `worker-deep` → `builtin:zai-coding-plan/GLM-5.3`，`thoughtLevel: max`
- `worker-review` → `builtin:zai-coding-plan/GLM-5.3`，`thoughtLevel: max`

本机唯一激活的 provider 是 `builtin:zai-coding-plan`，其 `GLM-5.3` 同时支持
`low`/`high`/`max` 三档思考等级，因此采用“同一模型 + 三档思考等级”的分层方式。
若要在其他机器使用，按 `plugins/zcode-dynamic-workflow/MODEL-MAPPING.md` 中的
步骤替换模型 ID 即可。

## 7. 来源快照

- 文档编制日期：2026-08-18
- OMC manifest 观察版本：`4.15.10`
- OMC 许可证：MIT
- ZCode 能力依据：官方 Plugin、Subagents、Skill、Hooks、Agent 文档

来源、兼容边界和许可证要求见 `docs/07-SOURCE-MAP.md`。
