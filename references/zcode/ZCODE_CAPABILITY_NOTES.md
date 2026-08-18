# ZCode 能力离线摘要

> 本文件是为了让无网络访问的本地 LLM了解本 MVP 所依赖的宿主能力。内容为官方文档的提炼，不是完整文档副本。快照日期：2026-08-18。

## Plugin

- 一个插件可以打包 Skill、Command、Subagent、MCP server 和 Hook。
- 推荐目录：

```text
plugin/
├── .zcode-plugin/plugin.json
├── commands/
├── skills/
├── agents/
├── hooks/hooks.json
└── .mcp.json
```

- Manifest 查找顺序优先 `.zcode-plugin/plugin.json`，并兼容 `.claude-plugin/plugin.json`。
- 本 MVP 只使用 `skills`、`commands` 和 `agents`。
- 本地 marketplace 可以通过 `marketplace.json` 指向一个相对插件目录。

## Skill

- 路径：`skills/<name>/SKILL.md`
- Frontmatter 至少包含 `name` 和 `description`。
- `description` 应明确写出使用时机。
- Skill body 按需加载，但所有启用 Skill 的元数据会进入模型上下文。
- Skill 过多会降低自动触发质量；本 MVP 只提供一个核心 Skill。

## Subagent

- Subagent 在隔离上下文中运行，结果返回主 Agent。
- ZCode 内置 `general-purpose` 和只读 `Explore`。
- 自定义 Agent 文件为 Markdown + frontmatter。
- 常用字段：
  - `name`
  - `description`
  - `model`
  - `thoughtLevel`
  - `tools`
  - `disallowedTools`
  - `maxTurns`
  - `injectAgentsMd`
  - `mcpServers`
- `thoughtLevel` 只有在指定具体 `model` 时生效；`inherit` 会跟随主 Agent。
- Agent description 会影响主 Agent自动选择。
- Subagent 不能生成自己的 Subagent。
- 多个前台 Subagent 可并行，主 Agent等待结果。
- 后台 Subagent 不阻塞主任务，结果会随后返回。

## Agent / AGENTS.md

- ZCode 工作区使用 `AGENTS.md` 作为稳定项目指令。
- 主 Agent负责目标、上下文、执行模式和最终任务推进。
- Subagent 默认可注入 AGENTS.md，但内置 Explore 是例外。
- 插件或 Agent 配置改变后，应新开 session。

## Hooks

- ZCode 支持 SessionStart、UserPromptSubmit、PreToolUse、PermissionRequest、PostToolUse、PostToolUseFailure 和 Stop。
- Hook 是本地子进程协议。
- 本 MVP 不需要 Hook；除非后续行为证据表明 Skill 无法稳定触发，否则不要添加。

## 官方来源

- https://zcode.z.ai/en/docs/agents
- https://zcode.z.ai/en/docs/plugin
- https://zcode.z.ai/en/docs/subagents
- https://zcode.z.ai/en/docs/skill
- https://zcode.z.ai/en/docs/hooks
- https://zcode.z.ai/en/docs/configuration
