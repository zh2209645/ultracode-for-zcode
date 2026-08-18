# ZCode Dynamic Workflow MVP：介绍与执行方案

## 摘要

本项目不是把 oh-my-claudecode 整体迁移到 ZCode，也不是复刻 Claude Code Dynamic Workflows 的 JavaScript runtime。首版只提炼一个最核心、最稳定的能力：

> **由主 Agent 动态规划工作，并根据每个子任务的难度、风险和独立性，选择不同性能层级的 ZCode subagent。**

ZCode 已经原生提供主 Agent、隔离 subagent、并行前台执行、后台执行、Skill、Command 和插件打包能力。首版因此不需要另建任务服务器、队列、Hook 状态机或 MCP 编排器。插件只需向主 Agent提供一套清晰的委派政策，并注册三个可被选择的通用 worker。

---

## 一、项目定位

### 1.1 要解决的问题

没有策略约束时，主 Agent 常见两类失误：

1. 对简单任务也调用强模型或多个 Agent，增加成本和协调开销；
2. 对复杂任务不拆解或使用过弱 Agent，导致反复失败、结果不完整或主上下文过载。

本插件通过“轻量决策协议”改善这两个问题。

### 1.2 核心用户体验

用户可以自然描述任务，也可以显式调用：

```text
/ultracode 重构认证模块，并补齐测试
```

主 Agent随后执行：

```text
理解目标
  ↓
拆分原子任务
  ↓
识别依赖与并行机会
  ↓
为每个任务评估难度
  ↓
直接完成 / Explore / worker-fast / worker-standard / worker-deep
  ↓
汇总结果
  ↓
必要时升级或重规划
  ↓
验证并报告
```

### 1.3 首版成功标准

- 简单任务不委派；
- 只读搜索优先使用内置 Explore；
- 常规任务优先使用 standard；
- 复杂、高风险任务使用 deep；
- 多个独立任务可并行；
- 有依赖任务按波次执行；
- subagent 不掌握总任务规划；
- worker 失败时只升级一次，不无限循环；
- 最终结论由主 Agent给出；
- 不依赖任何额外运行时。

---

## 二、首版范围

### 2.1 必须实现

```text
zcode-dynamic-workflow/
├── .zcode-plugin/plugin.json
├── commands/ultracode.md
├── skills/dynamic-workflow/SKILL.md
└── agents/
    ├── worker-fast.md
    ├── worker-standard.md
    └── worker-deep.md
```

### 2.2 明确不实现

- OMC Team pipeline；
- shared task list、worker registry、mailbox、heartbeat；
- Ralph persistence 或 stop-loop；
- Hook 注入；
- MCP orchestration；
- HUD、通知和 token 统计；
- worktree、自动合并；
- 独立任务数据库；
- nested agent delegation；
- 固定的 Plan → Execute → Verify 流水线；
- 自动生成 JavaScript workflow 脚本。

---

## 三、架构

### 3.1 控制面

主 Agent是唯一控制面，负责：

- 理解用户意图；
- 判断是否需要动态工作流；
- 创建临时任务图；
- 为任务评分；
- 决定 Agent 层级；
- 控制并行和依赖波次；
- 整合返回结果；
- 决定是否升级、补充或停止；
- 执行最终验证；
- 面向用户输出结论。

### 3.2 工作面

首版只有四种工作路径：

| 路径 | 用途 |
|---|---|
| 主 Agent 直接处理 | 极小、清晰、低风险任务 |
| 内置 Explore | 只读搜索、调用链、文件定位、证据收集 |
| worker-fast | 简单、机械、低耦合、低风险任务 |
| worker-standard | 常规实现、测试、文档和中等复杂修改 |
| worker-deep | 架构、复杂重构、根因分析、高风险改动、独立复核 |

### 3.3 无持久状态

任务图只存在于主 Agent 当前会话上下文中。插件不写入 `.omc/`、`.zcode-workflow/` 或自有数据库。只有明确承担文件写入的 worker 可按任务契约更新工作区 `.zcode/**` 中的 ZCode todo/任务日志；探索、分析和验证类 subagent 通过 ZCode 本身返回任务完成信息，不访问 `.zcode`。这些日志不得扩展为插件自有恢复状态。首版不支持中断恢复；长程持续执行由 ZCode 本身的任务和 Goal 能力承担，而不是本插件再实现一次。

---

## 四、动态路由算法

### 4.1 子任务评分

每个子任务按以下五个维度各计 0–2 分：

| 维度 | 0 | 1 | 2 |
|---|---|---|---|
| 范围 | 单点、单文件 | 2–5 文件或单模块 | 跨模块、跨系统 |
| 模糊度 | 目标明确 | 有少量未知 | 需求或根因明显不确定 |
| 耦合与推理 | 机械修改 | 需要常规推理 | 需要架构或长链推理 |
| 风险 | 文档/非行为 | 普通业务行为 | 安全、数据、权限、兼容性、生产配置 |
| 验证成本 | 单个检查 | 若干测试 | 多层测试、人工验证或回归面大 |

总分与默认路由：

| 总分 | 默认执行者 |
|---|---|
| 0–2 | 主 Agent直接完成 |
| 3–4 | worker-fast |
| 5–7 | worker-standard |
| 8–10 | worker-deep |

### 4.2 路由覆盖规则

以下规则优先于总分：

- 只读代码库搜索：内置 Explore；
- 安全、认证、授权、数据迁移、公开 API 兼容性：至少 deep 分析或 deep 复核；
- 架构决策、跨模块重构、连续失败后的根因分析：deep；
- 纯机械批量替换、格式修复、简单文档更新：fast；
- 一般功能实现、测试补充、常规 bug 修复：standard；
- worker 不能再派生 Agent；
- 主 Agent不应把“整个项目目标”直接丢给 worker。

### 4.3 并行规则

- 只有不存在写冲突、输出依赖或共享状态依赖的任务才能并行；
- 同时最多运行 3 个可写 worker；仅只读 Explore 可将 subagent 总并发提高到 10；
- 同一文件的多个写任务默认串行；
- 先探索再实现时，探索是 Wave 1，实现是 Wave 2；
- 实现依赖接口决策时，接口决策先完成；
- 多个独立目录或独立测试文件可以并行；
- 主 Agent需要立即使用结果时使用前台并行；
- 长时间、非阻塞调查可由主 Agent决定后台运行。

### 4.4 升级和停止

- fast 失败或发现范围扩大：升级到 standard；
- standard 失败、出现架构/高风险问题：升级到 deep；
- deep 失败：主 Agent重新审视任务边界、前置条件或用户约束；
- 同一子任务最多自动升级一次；
- 不做无限重试；
- 证据不足时报告 partial/blocked，而不是伪造完成。

---

## 五、模型层级

### 5.1 多模型环境

推荐：

- fast：小型、低延迟、低成本模型；
- standard：均衡的主力编码模型；
- deep：当前可用的最强推理/编码模型。

### 5.2 单模型环境

可以把同一个模型 ID 配置给三个 worker，并使用不同思考等级：

- fast：`low`
- standard：`high`
- deep：`max` 或该模型支持的最高等级

注意：ZCode 只有在 Agent 指定了具体模型 ID 时才应用独立 `thoughtLevel`；`model: inherit` 会跟随主 Agent，单独配置的 thought level 不生效。

---

## 六、从 OMC 提炼什么

### 6.1 保留的思想

来自 OMC `ultrawork` 和 agent tier 设计的核心思想：

- 不串行执行彼此独立的工作；
- 非简单任务先建立依赖感知的执行波次；
- 简单、标准、复杂任务使用不同模型层级；
- 委派结果必须简短、包含文件和验证状态；
- 复杂工作仍需最低限度验证；
- 将并行能力作为可组合能力，而不是持久运行时。

### 6.2 不迁移的内容

OMC 当前包含大量 skills、agents、MCP、CLI bridge、Team runtime、状态管理、HUD 和持久执行逻辑。首版全部不迁移。原因不是这些能力无价值，而是它们不属于本项目的最小目标，并会显著增加维护面和宿主耦合。

---

## 七、执行方案

### 阶段 0：基线

1. 在目标仓库创建独立分支。
2. 记录当前 ZCode 版本。
3. 确认插件、Skill 和 custom subagent 功能可用。
4. 记录三个真实模型 ID。
5. 保存一次未安装插件时的基线行为。

### 阶段 1：建立最小插件

1. 创建 `.zcode-plugin/plugin.json`。
2. 只声明 `skills`、`commands`、`agents`。
3. 不声明 Hook 和 MCP。
4. 创建本地 `marketplace.json`。
5. 在 ZCode 中安装并确认插件可见。

### 阶段 2：实现 Dynamic Workflow Skill

1. 明确触发条件；
2. 明确“不委派”的条件；
3. 写入五维评分和路由表；
4. 写入任务图、波次和并发规则；
5. 写入 worker 输入与返回契约；
6. 写入升级和停止条件；
7. 保持 Skill 简洁，避免复制完整 OMC harness。

### 阶段 3：实现三个 worker

每个 worker 只需要：

- 精确 description；
- 指定模型和 thoughtLevel；
- 限制最大 turns；
- 说明不接管总目标、不再委派；
- 说明只完成分配范围；
- 说明返回结果和验证证据。

### 阶段 4：显式命令

实现 `/ultracode`：

- 自动挂载 `dynamic-workflow` Skill；
- 将用户参数视为总任务；
- 提醒主 Agent保留规划和整合权；
- 不写死固定流水线。

### 阶段 5：静态验证

- JSON 可解析；
- YAML frontmatter 完整；
- Skill description 小于限制；
- Agent 的 `name`、`description`、`model`、`thoughtLevel` 可识别；
- 文件位置符合 ZCode 目录规范；
- 没有占位模型 ID 残留。

### 阶段 6：行为测试

至少覆盖：

1. 单行小改：主 Agent直接完成；
2. 只读定位：Explore；
3. 简单机械修改：fast；
4. 常规功能：standard；
5. 跨模块重构：deep；
6. 三个独立任务：并行；
7. 有依赖任务：分波次；
8. fast 失败：升级 standard；
9. 高风险认证修改：deep 分析或复核；
10. worker 返回不完整：主 Agent补充验证。

### 阶段 7：收敛

- 优先调整 Skill 和 Agent description；
- 删除未被行为测试证明有价值的提示；
- 不新增 runtime 来修复本可由提示解决的问题；
- 更新 README、模型映射和测试记录；
- 保留 MIT 许可证和上游致谢。

---

## 八、完成定义

首版只有在以下条件全部满足时才完成：

- 插件可从本地 marketplace 安装；
- Skill、Command、三个 Agent 均被 ZCode 识别；
- 三个 Agent 使用真实模型 ID；
- 路由测试表明主 Agent能区分直接、fast、standard、deep；
- 独立任务能并行，有依赖任务不会错误并行；
- subagent 不会再派生 subagent；
- 最终输出包含验证证据；
- 项目没有 Hook、MCP、额外 runtime 或持久状态；
- 许可证要求满足；
- 有一份可复现的测试记录。
