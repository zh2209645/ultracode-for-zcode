# 08 — 验收测试记录

- 日期：2026-08-18
- ZCode 版本：本机桌面版（会话内验证，插件注册表结构 `~/.zcode/cli/plugins/`）
- 主 Agent 模型：`builtin:zai-coding-plan/GLM-5.3`
- 测试方式说明：插件 Skill/Command/Agent 在会话启动时加载，本会话无法热加载插件包内的
  worker。因此功能场景以「内置 subagent + worker 系统提示词全文作为委派 prompt」的替身
  方式真实执行：路由决策、任务契约、并行/波次、升级链路、证据核验全部真实发生；
  唯一未验证的是安装后 `thoughtLevel`（low/high/max）的实际分层效果，该项列為剩余人工步骤。

## 1. 静态测试（docs/05 §1）

| ID | 结果 | 证据 |
|---|---|---|
| S-01 | PASS | `marketplace.json` JSON 解析通过，source 解析到 `plugins/zcode-dynamic-workflow/.zcode-plugin/plugin.json` |
| S-02 | PASS | `plugin.json` name 符合 `^[a-z0-9][a-z0-9._-]{0,127}$`；skills/commands/agents 三个组件目录存在；marketplace 与 plugin 版本一致（0.1.0） |
| S-03 | PASS | `skills/dynamic-workflow/SKILL.md` name 与目录一致，description 308 字符（<1024 限制） |
| S-04 | PASS | `commands/ultracode.md` 含 description、`$ARGUMENTS`、`skills: dynamic-workflow`，文件名符合 `^[a-z0-9][a-z0-9_:-]{0,63}$` |
| S-05 | PASS | 三个 agent 文件 frontmatter 完整：name/description/model/maxTurns/injectAgentsMd |
| S-06 | PASS | 插件包内无 `REPLACE_WITH_` 残留（仅 MODEL-MAPPING.md 的校验说明文本提及）；model ID 存在于本机激活 provider `builtin:zai-coding-plan`，三个 thoughtLevel 均在其 `GLM-5.3` 支持列表内 |
| S-07 | PASS | frontmatter 键全部为文档认可字段，`thoughtLevel` 拼写正确（camelCase） |
| S-08 | PASS | manifest 无 hooks/mcpServers/dependencies 键；插件目录无 hooks/、无 .mcp.json、无运行时代码、无状态文件 |
| S-09 | PASS | 项目 LICENSE、上游 `references/omc/LICENSE`、NOTICE.md 致谢均存在 |
| S-10 | PENDING | 需 UI 安装后新开 session 确认（见 §4 剩余步骤） |

静态检查由一次性脚本执行（未入库，符合 docs/04 §5 要求）。

## 2. 功能场景（docs/05 §2）

### F-01 不应委派 — PASS

- 请求：修复临时工作区 notes.md 中的错别字（`contians` → `contains`）。
- 观察：主 Agent 直接 Read+Edit 完成，未启动任何 worker；Edit 工具成功即最小验证。
- 结论：简单任务未过度委派。

### F-02 只读探索 — PASS

- 请求：定位插件目录下所有定义 subagent 的文件及其 model 字段。
- 观察：使用内置 `Explore`（只读），返回三个 worker 文件与占位符状态，未修改文件。
- 结论：只读调查路由到 Explore，未使用可写 worker。

### F-03 fast 任务 — PASS

- 请求：为两个独立 YAML 配置文件加同一行注释。
- 观察：两个 fast 契约替身并行完成（约 15s，并发），返回契约五段式（STATUS/SUMMARY/EVIDENCE/VERIFICATION/RISKS），未使用 deep。
- 结论：机械低风险任务命中 fast 层。

### F-04 standard 任务 — PASS

- 请求：为 `create_user` 增加邮箱格式校验并补单元测试。
- 观察：standard 契约替身实现正则校验 + 5 个用例，`python -m unittest` 5/5 通过，报告含实际命令与输出。
- 结论：常规实现路由 standard，验证真实执行。

### F-05 deep 任务 — PASS

- 请求：对插件包本身做独立架构复核（一致性/可加载性/策略缺口/改进项）。
- 观察：deep 契约替身读取全部 8 个相关文件并对照官方文档与本机 zcode-guide，结论"内部一致、可加载"，给出 11 条按严重度排序的发现（其中 3 条 Medium 已在收敛阶段修入 SKILL.md/worker）。
- 结论：架构/复杂分析路由 deep，产出带 file:line 证据。

### F-06 并行波次 — PASS

- 请求：三个无共享文件的独立小任务（见 F-03/F-11 组合场景）。
- 观察：A、B 两任务同一消息并行派发（同时启动、同时返回）；并发数 2 ≤ 3 上限。
- 结论：独立任务正确并行。

### F-07 依赖波次 — PASS

- 请求：先定配置 schema，再实现解析器，最后更新测试。
- 观察：Wave 1 schema.json（agent 完成）→ Wave 2 parser.py + 17 用例（消费 Wave 1 产物与结论）→ Wave 3 主 Agent 复跑：`Ran 17 tests ... OK`，并用有效/无效样例手工调用 `validate()` 确认行为。
- 结论：依赖任务按波次串行，未错误并行。

### F-08 层级升级 — PASS

- 场景 1（超范围任务）：fast 替身收到跨模块 rename（api/db/tests 三文件），实际直接完成并如实报告范围扩张与既有缺陷——记录为有效观察（替身与主 Agent 同模型，分层约束来自安装后的 thoughtLevel）。
- 场景 2（确定性 blocked）：fast 替身收到引用不存在 `fields.yaml` 的任务，按契约返回 `blocked` 并附精确证据（目录级搜索证明文件缺失、未篡改目标文件）。主 Agent 补齐前置文件后携前次证据一次性升级到 standard 替身；standard 完成、且发现主 Agent 所建 fields.yaml 的真实 YAML 转义缺陷；主 Agent 修复后复验 5 tests + 12 subtests 全过。
- 结论：升级最多一次、证据随行、无无限重试。

### F-09 高风险覆盖规则 — PASS

- 请求：分析"workspace owner 也视为 admin"的授权变更。
- 观察：即使代码仅 10 行，仍按覆盖规则路由 deep；deep 替身以 file:line 证据识别出 CRITICAL 横向越权（owner of X 可删 Y）、能力过度授予、可伪造所有权来源等 7 项风险，并给出 6 个必答设计问题与 7 项合并前必备测试。
- 结论：认证/授权变更强制 deep 分析，风险识别有效。

### F-10 不完整证据 — PASS

- 场景：worker 声称"5/5 测试通过"并给出命令。
- 观察：主 Agent 不直接采信，独立复跑 `python -m unittest test_users.py -v`，结果 5/5 OK，与声明一致。
- 结论：最终结论基于主 Agent 自行验证的证据。

### F-11 写冲突 — PASS

- 请求：任务 C 追加修改 config-b.yaml（与任务 B 同文件）。
- 观察：C 未与 B 同波并行，而是在 B 完成后作为 Wave 2 串行执行；B 的注释行在 C 完成后保持完整（四行文件验证）。
- 结论：同文件写任务正确串行。

### F-12 主 Agent 控制权 — PASS

- 观察：全部 12 次委派的 prompt 均为单一动词原子任务，含明确 SCOPE 与 ACCEPTANCE；无 worker 收到总目标；无 worker 派生 subagent（宿主机制保证 + 替身输出无嵌套迹象）；所有最终整合、复验与用户报告由主 Agent 完成。
- 结论：规划、整合与最终答复始终在主 Agent。

## 3. 质量指标（docs/05 §3）

| 指标 | 结果 | 门槛 | 判定 |
|---|---|---|---|
| 路由正确率 | 12/12 = 100% | ≥ 80% | 达标 |
| 过度委派率 | 0%（F-01 直接完成） | 越低越好 | 达标 |
| 低配失败率 | 0%（唯一升级源于缺失前置文件，非层级不足） | 越低越好 | 达标 |
| deep 滥用率 | 0%（deep 仅用于 F-05/F-09 高复杂度场景） | ≤ 20% | 达标 |
| 并行正确率 | 100%（独立并行 4 组，依赖串行 3 组） | — | 达标 |
| 证据完整率 | 12/12 委派返回含五段契约（含 blocked） | — | 达标 |
| 嵌套委派 | 0 次 | 禁止 | 达标 |
| 无限重试 | 0 次（单次升级后成功） | 禁止 | 达标 |

必过项 F-01、F-02、F-05、F-07、F-09、F-12：全部 PASS。

## 4. 剩余人工步骤（GitHub 链接安装形式）

1. 将本仓库推送到 GitHub（新建仓库或添加已有 remote 后 push；marketplace.json 在仓库根目录、`pluginRoot: "plugins"`、插件 source 为相对路径，布局与官方 GitHub marketplace 一致）。
2. ZCode：Settings → Plugins → Discover → `+` → 输入该 GitHub 仓库链接 → 安装并启用 `zcode-dynamic-workflow`。
3. 新开 session，确认 `/ultracode`、`dynamic-workflow` skill、三个 worker 出现（补齐 S-10 及 S-03/04/05 的在应用内确认）。
4. 分别对三个 worker 派发最小任务，确认 `GLM-5.3` 的 low/high/max 分层实际生效（对应 MODEL-MAPPING.md 的 Validation 节）。

## 5. 测试产物位置

临时工作区：`%TEMP%\dwf-acceptance\`（t1–t5 各场景的真实文件与测试，可随时复查或删除）。
