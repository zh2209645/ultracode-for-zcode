# 09 — 最终实施报告

- 日期：2026-08-19
- Plugin version：0.1.3（本地修复完成；远端 tag 待发布）
- ZCode 环境：Windows 桌面版，激活 provider `builtin:zai-coding-plan`（Z.ai Coding Plan）

## Implementation Summary

| 项 | 值 |
|---|---|
| Fast model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: low`（maxTurns 12） |
| Standard model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: high`（maxTurns 24） |
| Deep model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: max`（maxTurns 36） |
| Review model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: max`（maxTurns 30，只读） |

映射依据：本机唯一可用 provider 为 `builtin:zai-coding-plan`，其 `GLM-5.3` 注册的推理档位恰为
`low`/`high`/`max`（本机模型注册表与会话日志双重确认），因此采用 MODEL-MAPPING.md 的
Option B（同一模型、三档思考等级）——这是唯一完全被本机证据支撑的分层方案。
`glm-5-turbo`/`GLM-5.2` 等备选见 MODEL-MAPPING.md 备注及其前置验证条件。

## Files（初始实现历史）

> 本节仅记录 0.1.0 初始实现当时的文件变化；0.1.2 与 0.1.3 对这些文件的后续修改以各版本小节为准。

- 修改：
  - `marketplace.json`（安装格式修复：插件 source 由相对路径字符串改为
    `git-subdir` 对象形式 `{url, path, ref}`，指向 GitHub 仓库的
    `plugins/zcode-dynamic-workflow` 子目录；移除 `pluginRoot` 与 `strict`。
    根因：相对路径字符串仅适用于随应用内置的 marketplace；GitHub marketplace
    的在库子目录插件需用 `git-subdir`——与 claude-plugins-official 的实际
    可用条目及 zcode 运行时安装代码分支一致）
  - `plugins/zcode-dynamic-workflow/agents/worker-fast.md`（真实模型 ID；证据引用要求）
  - `plugins/zcode-dynamic-workflow/agents/worker-standard.md`（同上）
  - `plugins/zcode-dynamic-workflow/agents/worker-deep.md`（真实模型 ID）
  - `plugins/zcode-dynamic-workflow/skills/dynamic-workflow/SKILL.md`
    （收敛改进：fast 覆盖规则与"直接完成"的优先级澄清；冲突结果仲裁；无证据 `done`
    与中途截断的处理；部分失败对后续波次的约束；同层不重复派发；最终答复前回收后台结果）
  - `plugins/zcode-dynamic-workflow/MODEL-MAPPING.md`（占位符说明 → 实际映射与证据）
  - `plugins/zcode-dynamic-workflow/README.md`（安装步骤替代"先替换占位符"）
  - `README.md`（第 6 节改为已完成的模型映射）
  - `TASKS.md`（全部可验证项勾选）
  - `SHA256SUMS.txt`（重新生成）
- 新增：
  - `docs/08-TEST-RECORD.md`（12 个功能场景 + 10 项静态检查的完整记录）
  - `docs/09-FINAL-REPORT.md`（本文件）
- 删除：无
- 初始实现阶段未改动：`.zcode-plugin/plugin.json`、`commands/ultracode.md`、
  LICENSE、NOTICE.md、docs/01–07；其中 manifest 与 docs/01–07 已在后续安全修复和 reviewer 路由同步中修改。

## 0.1.2 Security Remediation

- 修改三个 `agents/worker-*.md`：关闭 AGENTS.md 注入；工具白名单统一为 Read/Edit/Write/Glob/Grep；移除 Bash/Web/MCP；仅写文件任务允许更新工作区 `.zcode/**` todo/log，探索/分析/复核/验证通过 ZCode subagent 返回。
- 修改 `skills/dynamic-workflow/SKILL.md`：写 worker 并发上限 3、Explore 总上限 10；命令和最终验证回到主 Agent；要求 Confirm Before Changes；不可信仓库禁止 Full Access 写委派。
- 修改 `marketplace.json`、`plugin.json` 与 Skill metadata：版本 0.1.2，source 固定 `v0.1.2`。
- 修改 README、MVP、docs/02/03/05/06/08/09：同步权限、`.zcode` 例外、验收和发布要求。
- 修改 `MODEL-MAPPING.md`：移除会导致占位符自检误报的字面量。
- 修改 `SHA256SUMS.txt`：按最终文件重新生成。
- 新增文件：无。
- 删除文件：无。

## 0.1.3 Read-only Reviewer

- 新增 `agents/worker-review.md`：Read/Glob/Grep 穷举白名单，GLM-5.3/max，关闭 AGENTS.md 注入，不访问 `.zcode`，只通过 ZCode result 返回。
- worker-review 使用高性能模型和最高推理档位，仅承接高风险代码/安全/权限/兼容性独立复核、裁决和验收证据结论；一般探索与普通分析分别交给 Explore 和主 Agent；三个原 worker 仅用于写任务。
- 高风险流程调整为 Explore 收集上下文 → reviewer 独立风险结论 → deep 写入 → 主 Agent测试 → reviewer 复核。
- marketplace/plugin/Skill 版本同步到 0.1.3，source 固定 `v0.1.3`。
- 新增 F-14 只读 reviewer 硬隔离验收场景。
- 新增根目录 `.gitattributes`，统一文本文件为 LF，确保 SHA-256 清单跨平台可复现。
- 新增根目录 `.gitignore` 排除 `.omo/` 本地审计工件，避免 `git add -A` 污染发布提交。

## Acceptance

- Static checks：0.1.3 配置、权限和结构检查 32/32 PASS；S-03/S-04/S-05/S-06 因 Skill、Command 和 reviewer 尚未在 0.1.3 新会话发现和调用而为 PARTIAL，S-10/S-12 因新版本尚未加载且 `v0.1.3` tag 尚未发布而为 PARTIAL。
- Functional scenarios：F-01–F-12 为 0.1.0 历史 PASS；由于本次修改了 Agent description、升级和权限规则，0.1.3 的强制回归 F-01/F-02/F-05/F-07/F-09/F-13/F-14 尚未实跑，整体保持 PARTIAL；旧 F-05/F-09 的 deep 只读分析证据不能替代当前 reviewer 路由。
- In-app confirmation（2026-08-19）：插件经 GitHub marketplace（git-subdir）安装启用；`/ultracode` 可用；主 Agent 成功派发 worker-fast subagent。插件 agent 不出现在 Settings → Subagents 属预期（该页仅列用户级 agent）。
- 如实记录的局限：功能场景中的 worker 以"内置 subagent + worker 系统提示词全文"替身执行（插件安装前）；
  low/high/max 的档位差异效果未做逐一量化对比（worker-fast 派发链路已验证，worker-review 需在 0.1.3 新 session 验证）。

## Scope Guard

- Hooks added: no
- MCP added: no
- Runtime code added: no（纯 JSON/Markdown 包；worker 运行时仅有文件工具）
- Persistent state added: no（任务图仅存在于会话上下文；写文件任务可更新 ZCode 工作区 `.zcode` todo/log，但插件不建立恢复数据库）

## Remaining Manual Steps

- 必须：提交并发布受保护或签名的 `v0.1.3` tag，使 marketplace 固定引用可解析且不可被普通 force-update 漂移。
- 必须：刷新 marketplace、更新插件并新开 session。
- 必须：按 docs/05 §5 重跑 0.1.3 强制回归 F-01/F-02/F-05/F-07/F-09/F-13/F-14，并记录当前 reviewer 路由证据。
- 必须：实跑一个写任务，确认只更新 SCOPE 文件与当前任务 `.zcode` todo/log；再跑 worker-review，确认其只有 Read/Glob/Grep、不访问 `.zcode`、只通过 ZCode subagent 返回结果。
- 必须：为 writer 和 reviewer 实跑 symlink/junction/reparse-point 穿透及 TOCTOU 并发替换场景，确认每次访问前重新解析，并拒绝所有越出可信工作区或 SCOPE 的访问；宿主不能原子绑定真实目标时任务 blocked。

## Risks

- `builtin:zai-coding-plan/GLM-5.3` 为本机限定映射；换机或换 provider 需按 MODEL-MAPPING.md 步骤重配。
- `model` 字段使用 provider 限定形式（日志中的规范形态）；若安装后 agent 无法调用模型，
  回退为裸 ID `GLM-5.3` 是文档中的第一调整项（docs/04 §4）。
- Skill 自动触发依赖 description 质量（当前已前载触发词）；若触发不稳，
  按docs/04 §7 顺序先调 description，不加 Hook。
- `SCOPE`、`.zcode` 和链接路径门禁属于 prompt/编排契约而非路径 sandbox；canonical 检查与访问之间仍有 TOCTOU 风险，主 Agent必须在每次访问前重新验证规范化真实目标并防止并发替换，实际写目标仍须由 Confirm Before Changes 或等价宿主权限逐项批准；宿主不能原子绑定目标时任务 blocked。
