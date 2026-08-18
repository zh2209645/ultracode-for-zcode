# 01 — 项目简介与范围

> EN: Project brief and scope — a ZCode-native delegation-policy plugin distilled from
> oh-my-claudecode; the first version ships one skill, one command, three write workers,
> and one read-only reviewer, with no hooks / MCP / runtime / persistent state.

## 项目代号

`zcode-dynamic-workflow`

## 背景

oh-my-claudecode 通过大量 Skills、Agents、Hooks、MCP 和 runtime 提供多 Agent 编排。它的 `ultrawork` 中最值得首版复用的部分，是“先判断任务是否适合并行，再按难度把任务路由到不同性能层级”的决策方式。

ZCode 已经原生支持：

- 主 Agent；
- 内置只读 Explore；
- 自定义 Subagent；
- 多个前台 Subagent 并行；
- 后台 Subagent；
- Skills、Commands 和 Plugin 打包。

因此，首版没有必要复制 OMC 的完整 harness。

## 产品目标

创建一个轻量 ZCode 插件，为主 Agent提供动态委派协议，使它在每次实质性任务中能够自主决定：

- 是否委派；
- 如何拆分；
- 哪些任务并行；
- 哪些任务有依赖；
- 每个任务使用什么性能层级；
- 何时升级；
- 如何整合和验证。

## 用户故事

### 用户故事 1：简单任务

作为用户，我提交一个单文件、明确、低风险修改时，主 Agent应直接处理，不应为了“多 Agent”而增加委派开销。

### 用户故事 2：并行任务

作为用户，我提交多个彼此独立的修改时，主 Agent应把它们放入同一执行波次，并根据难度选择不同 worker。

### 用户故事 3：复杂任务

作为用户，我提交跨模块重构时，主 Agent应先用 Explore 完成必要探索并建立任务依赖；普通分析由主 Agent完成，需要高价值独立风险结论时使用只读 worker-review，只有明确的复杂文件修改才交给 deep worker。

### 用户故事 4：成本控制

作为用户，我希望简单任务使用快速模型，常规任务使用均衡模型，只有复杂或高风险任务使用最强模型。

### 用户故事 5：结果可信

作为用户，我希望最终结论由主 Agent综合给出，并包含实际验证结果，而不是只转述 worker 的自我声明。

## 非目标

首版不解决：

- 任务跨会话恢复；
- 自动持续运行直到完成；
- 分布式 worker；
- 多 CLI 供应商协同；
- Agent 共享任务队列；
- 自动 worktree 和合并；
- 可视化运行面板；
- Agent 成本统计；
- 复杂策略配置 UI；
- 通用 workflow scripting runtime。

## 设计约束

- 主 Agent始终是唯一 orchestrator；
- ZCode Subagent 不能生成自己的 Subagent；
- 插件应仅由 JSON 和 Markdown 构成；
- 不依赖外部服务；
- 不假设具体工具调用语法；
- 模型 ID 必须由本地环境确定；
- 对 ZCode Beta 能力保持兼容边界说明；
- 提示词应短、明确、证据导向。

## Definition of Done

详见 `05-ACCEPTANCE-TESTS.md`。核心完成条件是：插件可加载、四个 Agent 的模型映射真实可用、主 Agent能够正确选择直接/Explore/review/fast/standard/deep，且没有引入额外 runtime。
