# 08 — 验收测试记录

- 日期：2026-08-18
- ZCode 版本：本机桌面版（会话内验证，插件注册表结构 `~/.zcode/cli/plugins/`）
- 主 Agent 模型：`builtin:zai-coding-plan/GLM-5.3`
- 测试方式说明：插件 Skill/Command/Agent 在会话启动时加载，行为测试当日的会话无法热加载插件包内
  worker。因此功能场景以「内置 subagent + worker 系统提示词全文作为委派 prompt」的替身
  方式真实执行：路由决策、任务契约、并行/波次、升级链路、证据核验全部真实发生。
  插件于 2026-08-19 安装启用后补充确认：新会话中 `/ultracode` 可用、主 Agent 可实际派发
  worker subagent（worker-fast 已验证），加载链路与替身测试一致。

## 1. 静态测试（docs/05 §1）

| ID | 结果 | 证据 |
|---|---|---|
| S-01 | PASS | `marketplace.json` JSON 解析通过；插件 source 为 `git-subdir` 对象形式（`url` + `path: plugins/zcode-dynamic-workflow`），与本机已验证可用的 claude-plugins-official 条目及 zcode 安装代码分支一致（相对路径字符串形式在 GitHub marketplace 安装时报 Unsupported source，已修复） |
| S-02 | PASS | `plugin.json` name 符合 `^[a-z0-9][a-z0-9._-]{0,127}$`；skills/commands/agents 三个组件目录存在；marketplace、plugin 与 Skill metadata 版本一致（0.1.3） |
| S-03 | PARTIAL | Skill 结构静态合规；0.1.0 在应用内确认插件启用后随会话加载并挂载 skill，但 0.1.3 尚需发布 tag、刷新 marketplace 并在新会话复验 |
| S-04 | PARTIAL | `commands/ultracode.md` 的 description、`$ARGUMENTS`、`skills: dynamic-workflow` 与文件名静态合规；`/ultracode` 仅有 0.1.0 历史可用证据，0.1.3 尚需新会话复验 |
| S-05 | PARTIAL | 四个 agent 文件的 frontmatter、`injectAgentsMd: false` 与穷举工具白名单静态验证通过；前三个写 worker 有历史派发证据，但 worker-review 尚需在 0.1.3 新会话确认被 ZCode 发现 |
| S-06 | PARTIAL | 插件包内无未解析模型占位前缀；model ID 与四个 thoughtLevel 均受本机 provider 配置支持，但 worker-review 尚未在 0.1.3 新会话实际调用 |
| S-07 | PASS | frontmatter 键全部为文档认可字段，`thoughtLevel` 拼写正确（camelCase） |
| S-08 | PASS | manifest 无 hooks/mcpServers/dependencies 键；插件目录无 hooks/、无 .mcp.json、无运行时代码、无状态文件 |
| S-09 | PASS | 项目 LICENSE、上游 `references/omc/LICENSE`、NOTICE.md 致谢均存在 |
| S-10 | PARTIAL | 0.1.0 安装启用后新会话正确加载（2026-08-19 确认）；0.1.3 必须在发布 `v0.1.3` tag 后刷新 marketplace 并新开 session 复验 |
| S-11 | PASS | 32 项 0.1.3 配置/权限检查通过：三个写 worker 仅有 Read/Edit/Write/Glob/Grep；worker-review 精确为 Read/Glob/Grep；四者均关闭 AGENTS.md 注入且无 Bash/Web/MCP；review 不具备任何写工具 |
| S-12 | PARTIAL | marketplace/plugin/Skill 版本均为 0.1.3，source 已固定 `v0.1.3`；远端 tag 尚待发布，因此远端可安装性未验证 |

静态检查由一次性脚本执行（未入库，符合 docs/04 §5 要求）。

## 2. 功能场景（docs/05 §2，0.1.0 历史证据）

本节记录初始版本的历史执行轨迹，其中 deep 承担只读分析/复核的旧路由已被 §10 的 0.1.3 reviewer 路由取代；这些记录不证明 worker-review 已在 0.1.3 新会话加载或通过 F-14。

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

## 4. 安装与识别确认（2026-08-19 完成）

1. ~~推送 GitHub~~（已完成：https://github.com/zh2209645/ultracode-for-zcode ，`git-subdir` source）
2. ~~UI 添加 marketplace 并安装启用~~（已完成：`zcode-dynamic-workflow@ultracode-for-zcode` 注册并启用，缓存含全部组件文件）
3. ~~新 session 确认可见性~~（已完成：`/ultracode` 可用、主 Agent 成功派发 worker-fast；插件 agent 不出现在 Settings → Subagents 属预期——该页仅列用户级 agent，用户确认无需处理）
4. ~~可选：对 worker-standard / worker-deep 也各派发一个最小任务，进一步确认 high/max 档位的实际效果~~（已完成 2026-08-19：三层 worker 均经插件本体实际派发成功，见 §6；档位差异的量化效果仍未测——单次实跑仅证明派发链路与任务完成，见 §6 诚实性说明）。

## 5. 测试产物位置

临时工作区：`%TEMP%\dwf-acceptance\`（t1–t5 各场景的真实文件与测试，可随时复查或删除）。

## 6. 插件本体实跑确认（2026-08-19，/ultracode 模式测试）

用户经 `/ultracode` 发起「ultracode模式测试」。主 Agent 按 dynamic-workflow skill 实跑一次完整
动态工作流；此前仅 worker-fast 经插件本体派发过（见 §4 第 3 条），本次补齐 standard/deep 两层。
主 Agent 模型 `builtin:zai-coding-plan/GLM-5.3`。

### 派发记录（worker 均经已安装插件派发，缓存路径
`C:\Users\qianp\.zcode\cli\plugins\cache\ultracode-for-zcode\zcode-dynamic-workflow\0.1.0`）

| Wave | 任务 | 执行者（层级） | STATUS | 关键结果 |
|---|---|---|---|---|
| 0 | 读验收文档/测试记录/模型映射，定位遗留项 | 主 Agent 直接 | — | 遗留项 = §4 第 4 条 |
| 1 | T1 安装缓存 vs 仓库 9 文件漂移审计 | 内置 Explore（只读） | 完成 | 9/9 SHA-256 一致；无占位符；plugin.json 无 hooks/mcpServers/dependencies；三个 worker frontmatter 均为具体 model + camelCase thoughtLevel |
| 1 | T2 SHA256SUMS.txt 逐项校验 | 插件 worker-fast（low） | done | 30/31 MATCH；发现自条目 = 空文件哈希（生成器缺陷，见下）；树干净、无清单外文件 |
| 1 | T3 两份 README 事实性声明审计 | 插件 worker-standard（high） | done | 唯一 STALE：根 README §3 包结构树漏列 5 个已跟踪文件；1 项 UNVERIFIABLE（provider 排他性属环境声明）；其余 SUPPORTED 带 file:line |
| 2 | T4 本实跑的路由合规独立复核 | 插件 worker-deep（max） | done | SKILL.md 各节合规 PASS；5 条发现（2 Medium / 3 Low）；判定 §4 第 4 条可关闭 |
| 3 | 本节记录写入 | 主 Agent 直接（按 T4 Finding D 由 standard 降级） | — | 单文件机械写入评分 0–2 → 直接完成，符合 SKILL.md §3 |
| 4 | git diff 复核 + 最终报告 | 主 Agent 直接 | — | 仅 docs/08 变更 |

- 波次与并发：Wave 1 三个独立任务前台并行（并发 3 = 上限）；Wave 2/3/4 因依赖前波结果串行。
- 升级 0 次；嵌套派生 0 次；无同文件并行写；worker 均返回五段契约（STATUS/SUMMARY/EVIDENCE/VERIFICATION/RISKS）。
- 派发链路证据（三重）：T1 全量 9 文件哈希比对一致；T4 独立复核 4 个政策文件（SKILL + 三 worker）哈希一致；worker-deep 自述其系统指令与 worker-deep.md 正文逐字一致。
- 场景对应：F-01（Wave 0 直接）、F-02（T1 Explore）、F-03（T2 fast）、F-05（T4 deep）、F-06（Wave 1 并行）、F-07（依赖波次）、F-12（主 Agent 保留拆解/评分/整合/最终答复）；F-04 仅覆盖路由层（standard 实际使用），实现+测试类任务与 F-08/F-10/F-11 未在本轮复现（已由 §2 替身测试覆盖）。

### 诚实性说明

本次实跑证明 fast/standard/deep 三层均可经插件本体派发并返回带证据结果。因三层共用同一模型 ID
（仅 thoughtLevel 不同），单次运行**不能**量化 low/high/max 的档位效果差异——本节仅主张派发链路
确认，不主张档位性能结论（AGENTS.md：没有证据时不声称验证通过）。

### 实跑发现（已于 §7 轮全部修复）

1. `SHA256SUMS.txt` 对自身的条目为空文件哈希 `e3b0c442…`（生成器在写入前计算）；30 个内容文件
   哈希全部正确。修复方式：重新生成清单时排除自条目或写入后回填（T2）。
2. 根 README §3 包结构树漏列 `NOTICE.md`、`SHA256SUMS.txt`、`plugins/zcode-dynamic-workflow/README.md`、
   `references/omc/README.md`、`references/omc/executor-notes.md`（T3）。
3. SKILL.md 文案歧义（T4，仅建议调文案、不加 runtime）：Medium-A 批量机械校验的 fast 评分带不清；
   Medium-B 需执行命令的只读任务（如哈希校验）的 Explore/worker 边界未定义；Low-C deep 触发词缺
   「对决策/测试结果的独立复核」；Low-E Explore 返回契约未在 §8 约定。（Low-D 即本节 Wave 3 降级，
   已当场采纳。）

## 7. 修复轮与并发调整（2026-08-19，第二轮 /ultracode 会话）

用户指令：修复 §6 发现的 3 类问题，并将 subagent 并发上限 3 → 10。

### 波次与路由

| Wave | 任务 | 执行者 | 结果 |
|---|---|---|---|
| 0 | 补读 AGENTS 必读文档 + 全库定位并发引用（含不含"并发"关键字的 §4.3"最多同时派发 3 个"） | 主 Agent 直接 | 6 个政策文件的 8 处数字定位完成 |
| 1 | 13 处编辑：SKILL.md ×5（并发 10 + A/B/C/E）、README §3 树、MVP 主文档、docs/02/03×2/05/06 | 主 Agent 直接（小而精确的文本修改，F-01 行为） | 全部落盘 |
| 2 | R1 一致性清查 + R2 政策复核 | 内置 Explore + 插件 worker-deep（并行，F-02/F-05 回归） | R1 5/5 PASS；R2 判 A/B/C/E 关闭、release-ready |
| 2.5 | 按 R2 建议两处打磨（评分刻度三描述符对齐 0–2；§2.6 护栏句"上限是天花板不是目标"） | 主 Agent 直接 | 完成 |
| 3 | 本节记录 + 插件缓存同步 + SHA256SUMS 重新生成 | 主 Agent 直接（依赖前序波次定稿，F-07 依赖链） | 完成 |
| 4 | sha256sum -c 终验 + git status 复核 | 主 Agent 直接 | 31/31 OK（证据见会话最终报告） |

### 变更清单

1. 并发上限 3→10（用户指令）：`SKILL.md` §2.6、`ZCODE_DYNAMIC_WORKFLOW_MVP.md` §4.3、
   `docs/02` §5、`docs/03` §8 + §13 伪代码、`docs/05` F-06 预期、`docs/06` 设计要求 7。
   历史记录（`TASKS.md` 勾选项、本文档 §2/§6）保留"3"为史实，不改写历史。
   ZCode 能力 notes 未记载宿主强制并发上限（references/zcode §Subagent："多个前台 Subagent 可并行"），
   10 为政策默认值，无事实冲突；R2 风险表评估各风险均被现有文本缓解。
2. SKILL.md 歧义修复（§6 发现 3，全为文案）：A = fast 覆盖规则补"批量机械校验"（如逐条清单/哈希
   校验）且验证成本刻度补"一批机械逐项检查"；B = 只读覆盖行拆分：探索/映射/取证 → Explore，
   需执行命令的规则性校验（逐条检查、测试、构建）→ worker（即使零写入）；C = 新增 deep 触发
   "对主 Agent 重大决策/重要变更/验收结果的独立复核"；E = §8 补 Explore 结论也须带 file:line
   或命令证据。
3. README §3 包结构树补齐 5 个漏列文件；R1 逐叶比对确认树与 `git ls-files` 32 个跟踪文件完全一致
   （修复 §6 发现 2）。
4. `SHA256SUMS.txt` 重新生成并排除自条目，消除"自条目 = 空文件哈希"缺陷（修复 §6 发现 1）；
   本清单在全部内容修改定稿后生成。
5. 插件缓存 `…\cache\ultracode-for-zcode\zcode-dynamic-workflow\0.1.0\skills\dynamic-workflow\SKILL.md`
   同步为仓库版本，新会话即生效；已运行会话需新开 session（ZCode 能力 notes：配置改变后应新开
   session）。

### 回归（docs/05 §5：并发上限变更后至少重跑 F-01/02/05/07/09）

- F-01 PASS（实跑）：本轮 15 处小编辑全部主 Agent 直接完成，零委派。
- F-02 PASS（实跑）：R1 只读清查，全部结论带命令与 file:line 证据。
- F-05 PASS（实跑）：R2 deep 独立复核逐条裁决 + 全文件一致性 + 跨文档一致性 + 风险表。
- F-07 PASS（实跑）：编辑 → 复核 → 打磨 → 记录 → 清单生成严格按依赖波次串行，清单生成置于终位。
- F-09 未复现（无适用对象）：本仓库无认证/授权代码，安全覆盖行（SKILL.md §3）未改动，R2 复核
  确认其原文完好且与新增行无冲突；§2 F-09 历史 PASS 仍立。

### 遗留与建议

- R2 非阻断分歧 1 处：SKILL 将 Explore 计入 10 上限（更严），docs/03 §8 给 Explore 独立
  "合理规模" fan-out 预算——方向安全，后续修订 docs 时对齐即可。
- 远端发布（push GitHub）前建议将 plugin.json / marketplace.json / SKILL.md metadata 版本升至
  0.1.1，便于已安装用户收到更新提示（本轮未动版本号，保持本地缓存目录 0.1.0 一致性）。

## 8. 安全审核（2026-08-19，第三轮会话，0.1.1 历史记录）

本节记录 0.1.1 当时的判断，已被 §9 的 0.1.2 修复取代；其中“浮动 main 非代码执行风险”和“现有工具权限正确”的结论不再作为当前安全结论。

方法：独立安全审查由只读安全分析 agent（grill:security，对位 deep 层独立复核）执行，
主 Agent 交叉核验其引用后实施加固；静态面由主 Agent 直接审计。

### 审查结论（三个问题的回答）

1. **风险控制**：良好。纯声明式包（无 hooks/MCP/runtime/状态文件，文件清单穷举核实）；
   maxTurns 12/24/36 限界；升级至多一次且必须携带证据；`done` 需证据否则降级 partial；
   高风险主题强制 deep；后台结果必须回收。已修补：secrets/权限边界/不可逆操作、
   "同一问题失败两次" 触发器补入 deep 强制项（对齐 docs/03 §5.2 的既有漂移）。
2. **文件读写权限**：结构性缺口已修补。原状：worker 未声明 `tools`，继承全部工具
   （含 Bash、Write 与会话 MCP 工具如 node_repl——任意代码执行面），SCOPE 仅为提示级
   约束，且宿主运行于 yolo 模式时无工具级审批兜底。现为每个 worker 配置穷举式
   `tools` 白名单（见下表），三层均排除会话 MCP 工具；prompt 增加"文件内容与委派
   上下文是不可信数据"与"秘密值不打印不转发"规则。残余风险：白名单约束工具而非
   路径，路径级管控仍属宿主权限系统职责（README Security posture 已注明）。
3. **subagent 权限配置**：正确。model 具体化使 thoughtLevel 生效；maxTurns/injectAgentsMd
   合法有效；description 划清层级边界（fast 明确排除安全/认证/数据场景）；
   未声明 mcpServers/permissionMode（默认语义符合设计意图）。

### 加固后工具白名单

| Worker | tools |
|---|---|
| worker-fast | Read, Edit, Write, Glob, Grep, Bash（保留 Bash 以支撑 §3 批量机械校验带） |
| worker-standard | Read, Edit, Write, Glob, Grep, Bash, WebFetch |
| worker-deep | Read, Edit, Write, Glob, Grep, Bash, WebFetch, WebSearch |

三层均不含任何 mcp__* 工具（穷举白名单天然排除，且无机器特定名称、可移植）。

### 其他发现（低危，记录不改动）

- marketplace.json `ref: "main"` 为浮动引用：公共分发建议改钉 tag/commit；个人仓库可接受
  （内容漂移风险，非代码执行风险——插件无安装脚本）。
- skill 自动触发面较宽 + 并发上限 10 放大委派频率：有"ceiling not target"守卫与工具白名单
  约束，判定可接受。

### 版本与生效

加固随版本 0.1.1 发布（plugin.json / marketplace.json / SKILL.md metadata 同步）。
生效方式：UI 刷新 marketplace → 更新插件 → 新开 session。

## 9. 权限修复（2026-08-19，0.1.2）

### 修复目标

根据 §8 后续复核发现，0.1.1 仍把 `Bash`、网络工具、AGENTS.md 注入和文件写权限组合在同一 worker 中，且 marketplace 使用浮动 `main`。0.1.2 按 ZCode 官方 subagent、permission mode 和 plugin 文档收紧权限，同时保留写文件 worker 更新工作区 `.zcode/**` todo/任务日志的能力。

### 已实施控制

1. 三个 worker 全部设为 `injectAgentsMd: false`，主 Agent只传入审查后的最小可信项目规则。
2. 三个 worker 的穷举白名单统一为 `Read, Edit, Write, Glob, Grep`；移除 `Bash`、`WebFetch`、`WebSearch`，并继续排除所有 MCP 工具。命令、网络访问、测试和构建由主 Agent执行。
3. 只有明确承担文件写入的任务可以读写工作区相对路径 `.zcode/**`，且仅限当前任务 todo/log。Explore、分析、复核、验证任务不访问 `.zcode`，通过 ZCode subagent 返回结果。用户级 `~/.zcode`、缓存、凭据、其他 session 日志与插件自有恢复数据库均禁止。
4. 可写 worker 并发上限从 10 收紧为 3；仅只读 Explore 可以把 subagent 总并发提高到 10。共享 `.zcode` todo/log 文件串行更新，并行 worker 使用不同日志文件。
5. 写委派要求 `Confirm Before Changes` 或等价受限 sandbox；不可信仓库禁止在 `Full Access` 下派发可写 worker。
6. marketplace/plugin/Skill 版本同步到 0.1.2，远端 source 固定到 `v0.1.2` tag；发布时必须保护或签名该 tag；删除插件包内自引用的模型占位前缀文本。

### 静态验证

- PowerShell 结构化检查：30/30 PASS。
- 覆盖：三处版本、固定 ref、三个 agent 的注入开关/精确工具表/无 Shell-Web-MCP、`.zcode` 写任务门禁、只读任务通过 ZCode 返回、用户级 `.zcode` 禁止、3/10 双层并发、Confirm Before Changes、Full Access 禁止、无 Hook/MCP/runtime/executable、无模型占位前缀。
- JSON 解析与 `git diff --check`：PASS。

### 未完成的外部验证（0.1.2 历史状态）

- 0.1.2 当时的状态：`v0.1.2` tag 尚未发布，因此 S-10（0.1.2 新会话加载）与 S-12（远端 tag 可安装性）保持 PARTIAL；当前 0.1.3 门禁见 §10 和最终报告。
- 发布后需：刷新 marketplace → 更新插件 → 新开 session → 分别派发一个写任务和一个验证任务，确认前者可更新工作区 `.zcode` todo/log，后者只通过 subagent 结果返回且不写 `.zcode`。

## 10. 只读 reviewer 隔离（2026-08-19，0.1.3）

ZCode 官方文档确认内置 subagent 只有 `general-purpose` 和只读 `Explore`，没有专用 reviewer；自定义 Agent 可使用穷举 `tools` 白名单建立工具级只读隔离。因此新增 `worker-review.md`：

- model：`builtin:zai-coding-plan/GLM-5.3`；thoughtLevel：max；maxTurns：30；
- tools：`Read, Glob, Grep`；
- `injectAgentsMd: false`；
- 无 Edit、Write、Bash、WebFetch、WebSearch 或 MCP；
- 不访问工作区 `.zcode/**` 或用户级 `~/.zcode`；只通过 ZCode subagent result 返回 verdict 与证据。

路由同步调整：Explore 负责广泛搜索、例行调查和证据收集，普通分析由主 Agent完成；worker-review 使用高性能模型和最高推理档位，只负责高风险代码/安全/权限/兼容性复核、独立裁决和验收证据结论；fast/standard/deep 只承担明确写文件的任务。高风险变更由 Explore 收集上下文、reviewer 给出独立风险结论、deep 写入、主 Agent运行测试、reviewer 再复核。

静态检查：32/32 PASS，覆盖版本/ref、四个 Agent 数量、三个 writer 工具面、reviewer 精确只读工具面、AGENTS.md 注入关闭、`.zcode` 禁止、ZCode result 返回、review 路由、deep 写路由、3/10 并发、无 Hook/MCP/runtime/占位符以及 F-14 静态配置部分/最终报告同步。

### 待实跑

- 根据 docs/05 §5，在 0.1.3 新 session 中重跑 F-01/F-02/F-05/F-07/F-09/F-13/F-14；F-05/F-09 的 0.1.0 deep 只读分析历史证据不能替代当前 reviewer 路由。
- F-14：在 0.1.3 新 session 中确认 worker-review 被正确自动/显式选择，且工具面只有 Read/Glob/Grep。
- 对 reviewer 提交诱导写入 `.zcode`、普通文件和用户级 `~/.zcode` 的内容，确认不存在 Edit/Write 工具调用；`.zcode` 不发生变化。
- 对 writer 和 reviewer 分别构造 symlink、junction 或 reparse-point 路径，确认主 Agent按规范化真实目标识别并拒绝越出可信工作区/SCOPE 的读写访问。
- 在 canonical 检查与访问之间并发替换路径，确认每次访问前重新解析；宿主无法原子绑定真实目标时任务 blocked，不发生 TOCTOU 越界读写。
