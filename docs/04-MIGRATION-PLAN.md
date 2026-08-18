# 04 — 迁移与重构执行方案

## 1. 迁移策略

本项目采用“概念提炼 + ZCode 原生重写”，不采用“OMC 全量兼容层”。

### 上游输入

重点参考：

- OMC `skills/ultrawork/SKILL.md`
- OMC `docs/shared/agent-tiers.md`
- OMC 通用 executor 的范围和验证原则
- OMC MIT License
- ZCode 官方 Plugin、Subagents、Skill、Agent 文档

### 输出

一个纯 JSON/Markdown 的 ZCode 插件。

## 2. 提取矩阵

| OMC 内容 | 首版处理 | 原因 |
|---|---|---|
| Ultrawork 并行原则 | 重写并保留 | 与目标直接相关 |
| LOW/MEDIUM/HIGH tier | 简化为 fast/standard/deep | 减少角色数量 |
| Dependency-aware waves | 保留 | Dynamic workflow 核心 |
| Explicit model routing | 保留 | 插件主要目的 |
| Executor scope discipline | 精简保留 | 防止 worker 扩大范围 |
| Lightweight verification | 保留 | 防止虚假完成 |
| Team runtime | 删除 | ZCode 主 Agent原生编排 |
| Ralph persistence | 删除 | 不属于首版 |
| Hooks | 删除 | Skill/Command 足够 |
| MCP | 删除 | 无新工具需求 |
| HUD/telemetry | 删除 | 无关核心目标 |
| 32 个专业 Agent | 删除 | 高性能 LLM 不需要过细角色 |
| CLI bridge/providers | 删除 | 不做跨供应商调度 |
| OMC state files | 删除 | MVP 无持久状态 |

## 3. 文件级实施步骤

### 3.1 `marketplace.json`

目标：允许 ZCode 从本地目录加载插件。

检查：

- `pluginRoot` 指向 `plugins`；
- 插件 source 指向 `./zcode-dynamic-workflow`；
- marketplace 与 plugin 的版本一致；
- 版本变更时同步更新。

### 3.2 `.zcode-plugin/plugin.json`

只包含：

- name；
- version；
- description；
- author；
- repository/homepage（可选）；
- license；
- keywords；
- skills；
- commands；
- agents。

禁止声明：

- hooks；
- mcpServers；
- dependencies，除非后续有明确必要。

### 3.3 `skills/dynamic-workflow/SKILL.md`

实施要求：

1. `name` 与目录一致；
2. `description` 明确写出触发条件；
3. 不依赖 magic keyword hook；
4. 包含直接执行、Explore、三档 worker 的选择规则；
5. 包含五维难度评分；
6. 包含波次、并发上限和写冲突规则；
7. 包含 prompt/return 契约；
8. 包含升级和停止条件；
9. 不硬编码具体模型 ID；
10. 不引用 Claude Code 专用 Task 调用语法。

### 3.4 `commands/ultracode.md`

只做一件事：挂载 Skill 并传递 `$ARGUMENTS`。

不得：

- 在命令正文中重复整份 Skill；
- 写死固定流水线；
- 假设每个请求都必须并行；
- 强制使用 deep。

### 3.5 三个 Agent 文件

共同要求：

- `name`、`description`、`model`；
- 具体模型时配置合法 `thoughtLevel`；
- `maxTurns` 为正整数；
- 不允许 nested delegation；
- 不接管用户总目标；
- 最小范围；
- 返回证据；
- 不确定时 `partial` 或 `blocked`；
- 避免长篇角色设定。

差异：

| Agent | 推荐 maxTurns | 重点 |
|---|---:|---|
| worker-fast | 8–12 | 快速、机械、低风险 |
| worker-standard | 16–24 | 常规实现和验证 |
| worker-deep | 24–40 | 复杂推理、架构、高风险、复核 |

## 4. 模型映射步骤

1. 打开 ZCode 模型设置或读取本机已连接模型配置；
2. 记录真实 model id，而不是只记录 UI 显示名；
3. 选择 fast、standard、deep；
4. 检查各模型支持的 thought levels；
5. 替换 Agent 文件占位符；
6. 新开 ZCode session，使配置生效；
7. 分别显式调用三个 Agent 做最小任务；
8. 记录是否成功调用、耗时和输出质量。

若只有一个模型：

1. 三个 Agent 使用同一 model id；
2. fast 使用最低有效 thoughtLevel；
3. standard 使用日常高质量档；
4. deep 使用最高档；
5. 若模型不支持多 thought levels，则 fast/standard/deep 可以暂时通过 maxTurns 和 prompt 约束区分，但应在文档中标明这不是完整性能隔离。

## 5. 静态验证

建议实施者编写临时检查脚本或直接使用现有工具验证，但不要把脚本作为插件 runtime 依赖。

检查项：

- 所有 JSON 可解析；
- YAML frontmatter 有闭合分隔线；
- Skill description 不超限；
- Agent 无未知字段拼写；
- `thoughtLevel` 使用 camelCase；
- 不残留 `REPLACE_WITH_*`；
- 目录平铺符合 ZCode plugin 发现规则；
- 许可证文件存在。

## 6. 安装验证

1. 在 ZCode 打开任意 workspace；
2. Settings → Plugins；
3. Create → Add marketplace；
4. 选择本包根目录或 `marketplace.json`；
5. 安装并启用插件；
6. 检查插件详情中的 Skill、Command 和 3 个 Subagent；
7. 新开 session；
8. 使用 `/ultracode` 执行测试任务。

## 7. 行为调优顺序

如果主 Agent不正确委派，按以下顺序调整：

1. Agent `description`；
2. Skill 的触发 description；
3. 路由覆盖规则；
4. 示例；
5. worker system prompt；
6. 并发上限。

不要第一时间增加 Hook、脚本或状态机。

## 8. 推荐提交序列

```text
chore: add minimal zcode plugin marketplace
feat: add dynamic workflow delegation skill
feat: add fast standard and deep workers
feat: add ultracode command
docs: add model mapping and usage guide
test: record delegation acceptance scenarios
chore: add upstream attribution and license
```

## 9. 回滚策略

因为插件不写持久状态，回滚非常简单：

- 禁用或卸载插件；
- 删除本地 marketplace；
- 恢复 Agent 模型映射文件；
- 不需要迁移数据库或清理后台进程。

## 10. 最终报告模板

```text
## Implementation Summary
- Plugin version:
- ZCode version:
- Fast model:
- Standard model:
- Deep model:

## Files
- Added:
- Modified:
- Removed:

## Acceptance
- Static checks:
- Functional scenarios:
- Failed scenarios:

## Scope Guard
- Hooks added: no
- MCP added: no
- Runtime code added: no
- Persistent state added: no

## Remaining Manual Steps
- ...

## Risks
- ...
```
