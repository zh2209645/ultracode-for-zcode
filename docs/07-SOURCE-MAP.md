# 07 — 来源映射、兼容边界与许可证

> EN: Source map — snapshots of the upstream oh-my-claudecode material consulted, the
> compatibility boundary (ideas only, no upstream code copied), and MIT license /
> attribution obligations preserved in LICENSE, references/omc/LICENSE, and NOTICE.md.

## 1. 快照信息

编制日期：2026-08-18

### oh-my-claudecode

- 仓库：`https://github.com/Yeachan-Heo/oh-my-claudecode`
- 观察到的 plugin manifest 版本：`4.15.10`
- 许可证：MIT
- 核心参考：
  - `skills/ultrawork/SKILL.md`
  - `docs/shared/agent-tiers.md`
  - `.claude-plugin/plugin.json`
  - `agents/executor.md`
  - `LICENSE`

### ZCode

官方参考：

- Agent：`https://zcode.z.ai/en/docs/agents`
- Plugin：`https://zcode.z.ai/en/docs/plugin`
- Subagents：`https://zcode.z.ai/en/docs/subagents`
- Skill：`https://zcode.z.ai/en/docs/skill`
- Hooks：`https://zcode.z.ai/en/docs/hooks`
- Models：`https://zcode.z.ai/en/docs/configuration`

## 2. 关键事实

本方案依赖以下 ZCode 能力：

1. 插件可打包 Skills、Commands、Subagents、MCP 和 Hooks；
2. 推荐 manifest 位于 `.zcode-plugin/plugin.json`；
3. 插件 Agent 定义位于 `agents/*.md`；
4. Agent 文件支持 `model`、`thoughtLevel`、`maxTurns` 等字段；
5. 主 Agent根据 Agent description 判断何时自动调用；
6. 多个前台 Subagent 可并行运行；
7. 后台 Subagent 可在主任务继续时返回结果；
8. Subagent 不能生成自己的 Subagent；
9. 内置 Explore 是只读代码库搜索角色；
10. Skill 是按需加载的工作指令；
11. 过多 Skill 会稀释元数据预算，因此 MVP 只保留一个核心 Skill；
12. Plugin/Agent 配置更改通常需要新开 session 才能可靠生效。

## 3. OMC 到 MVP 的映射

| OMC 概念 | MVP 对应 |
|---|---|
| ultrawork | `dynamic-workflow` Skill |
| LOW tier | `worker-fast` |
| MEDIUM tier | `worker-standard` |
| HIGH tier | `worker-deep` |
| code/security reviewer | `worker-review`（只读） |
| explore agent | ZCode 内置 Explore |
| orchestrator | ZCode 主 Agent |
| Task graph / waves | 主 Agent临时上下文中的动态计划 |
| lightweight verification | 主 Agent最终验证 |
| model routing table | Skill 中的难度评分与覆盖规则 |

## 4. 未迁移的上游路径

本方案明确不需要读取或移植整个：

- `src/team/`
- `src/hooks/`
- `src/mcp/`
- `src/hud/`
- `src/providers/`
- `src/ralphthon/`
- `bridge/`
- `.mcp.json`
- OMC CLI 和 setup runtime
- 大量专业 Agent 和兼容 facade

本地 LLM若发现这些目录，应将其视为“非目标”，而不是待迁移列表。

## 5. 许可证处理

OMC 的 MIT License 允许使用、复制、修改、合并、发布和分发，但要求在软件的副本或实质性部分中保留版权和许可声明。

实施建议：

- 项目保留自己的 `LICENSE`；
- 在 `NOTICE.md` 或 README 中致谢 OMC；
- 若直接复制上游 Skill 或 Agent 大段文本，保留 `references/omc/LICENSE`；
- 更推荐重写简洁的 ZCode 原生提示，只继承概念；
- 不宣称本项目为 OMC 官方端口或官方支持版本。

## 6. 兼容边界

- ZCode custom subagent 能力当前可能仍处于 Beta，字段和行为未来可能变化；
- Plugin Agent 在 UI 中可能是只读的，因此模型 ID 最好在分发前写入文件；
- 模型 ID 和可用 thought level 取决于本机供应商配置；
- `model: inherit` 时独立 `thoughtLevel` 不生效；
- Agent/Plugin 修改后应新开 session；
- 本方案不依赖 Hook，因此避开 Hook 协议差异；
- 本方案不硬编码底层 Agent tool 调用语法，以降低版本耦合。

## 7. 更新检查

若实施日期晚于本快照，优先重新核对：

1. ZCode Plugin manifest 字段；
2. Agent frontmatter 字段名；
3. Subagent 是否仍禁止 nested delegation；
4. 前台并行和后台行为；
5. Skill description 和大小限制；
6. OMC 上游许可证；
7. 本机实际模型 ID 和 thought level。
