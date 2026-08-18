# 02 — MVP 架构

## 1. 总体架构

```text
User Request
    |
    v
ZCode Primary Agent
    |
    +-- Understand goal and constraints
    +-- Build temporary task graph
    +-- Score each task
    +-- Decide direct / delegate
    |
    +-- Direct work
    +-- Built-in Explore
    +-- worker-fast
    +-- worker-standard
    +-- worker-deep
    |
    v
Aggregate results
    |
    +-- Verify
    +-- Escalate once if needed
    +-- Re-plan when assumptions change
    |
    v
Final answer
```

## 2. 组件职责

### 2.1 Dynamic Workflow Skill

负责教主 Agent：

- 判断任务是否值得委派；
- 将总任务拆成原子任务；
- 建立依赖波次；
- 给任务评分；
- 选择 worker 层级；
- 控制并发；
- 形成高质量的委派 prompt；
- 整合 worker 输出；
- 升级失败任务；
- 完成最终验证。

Skill 不执行代码，也不维护状态。

### 2.2 `/ultracode` Command

显式入口。它只负责挂载 Skill 并把用户参数传给主 Agent，不负责生成固定任务 DAG。

### 2.3 worker-fast

适合：

- 明确、局部、机械任务；
- 小型文档或配置更新；
- 简单测试补充；
- 低风险批量修改；
- 主 Agent已给出准确文件范围和验收条件的任务。

### 2.4 worker-standard

适合：

- 常规功能实现；
- 中等范围 bug 修复；
- 常规测试和文档；
- 需要理解现有模式但不涉及架构改变的任务。

### 2.5 worker-deep

适合：

- 架构设计；
- 跨模块重构；
- 高风险安全、权限、数据和兼容性问题；
- 根因不明的复杂故障；
- 对重要实现进行独立复核。

### 2.6 内置 Explore

不重复实现 Explore Agent。使用 ZCode 内置只读角色完成：

- 文件定位；
- 调用链调查；
- 实现入口和依赖映射；
- 风险和证据收集；
- 并行搜索。

## 3. 控制权边界

### 主 Agent拥有

- 用户目标；
- 完整上下文；
- 临时任务图；
- 并行和依赖关系；
- 接受标准；
- 整合决策；
- 最终验证；
- 最终答复。

### Worker 只拥有

- 被分配的原子任务；
- 为完成该任务所需的局部上下文；
- 明确的允许范围；
- 明确的返回格式。

Worker 不应：

- 扩大总范围；
- 重新解释用户目标；
- 调度其他 worker；
- 自行宣布整个项目完成；
- 修改与任务无关的文件。

## 4. 状态模型

首版为无状态策略插件：

```text
Persistent plugin state: none
Persistent task graph: none
Cross-session resume: none
External database: none
```

动态任务图只存在于当前主 Agent上下文。只有文件写入任务的 worker 可以更新工作区 `.zcode/**` 中由 ZCode 使用的 todo/任务日志；探索、分析和验证类 subagent 通过 ZCode 本身返回结果，不访问 `.zcode`。插件不建立自己的恢复状态或任务数据库。这样可以降低实现和维护成本，并最大程度利用 ZCode 的原生长任务能力。

## 5. 并发模型

ZCode 可让多个前台 Subagent 并行，主任务等待它们全部返回；后台 Subagent 则允许主任务继续。Skill 不应硬编码底层调用形式，只需定义选择原则：

- 需要结果才能继续：前台；
- 长时间且不阻塞当前路径：后台；
- 可写 worker 并发上限：3；
- 仅只读 Explore 可将 subagent 总并发提高到 10；
- 写同一文件：默认不并行；
- 不确定是否独立：先串行或先 Explore。

## 6. 模型映射

### 推荐方案 A：多模型

```text
worker-fast     -> fast/cheap model, low thinking
worker-standard -> balanced coding model, high thinking
worker-deep     -> strongest model, max or highest thinking
```

### 推荐方案 B：单模型三档

```text
worker-fast     -> same model, low
worker-standard -> same model, high
worker-deep     -> same model, max
```

具体模型 ID 必须来自本机 ZCode 模型列表，不应在通用 Skill 中硬编码。

## 7. 为什么不使用 Hook

首版不需要：

- SessionStart 注入：Skill 和命令已经足够；
- UserPromptSubmit 关键词检测：自然触发和 `/ultracode` 足够；
- Stop gate：不实现持久循环；
- Pre/Post tool enforcement：主 Agent和 worker prompt 已覆盖范围。

只有在行为测试明确证明 Skill 无法稳定触发时，才应在后续版本评估 Hook；它不是 MVP 依赖。

## 8. 为什么不使用 MCP 或 runtime

MCP 和运行时代码适合提供新工具或实现宿主缺失能力。本项目所需能力——主 Agent规划、Subagent、并行、Skill、Command——ZCode 已原生提供。额外 runtime 只会增加：

- 安装复杂度；
- 跨平台问题；
- 安全审查面；
- 版本耦合；
- 维护成本。
