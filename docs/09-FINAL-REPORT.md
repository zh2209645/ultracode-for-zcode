# 09 — 最终实施报告

- 日期：2026-08-18
- Plugin version：0.1.0
- ZCode 环境：Windows 桌面版，激活 provider `builtin:zai-coding-plan`（Z.ai Coding Plan）

## Implementation Summary

| 项 | 值 |
|---|---|
| Fast model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: low`（maxTurns 12） |
| Standard model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: high`（maxTurns 24） |
| Deep model | `builtin:zai-coding-plan/GLM-5.3` @ `thoughtLevel: max`（maxTurns 36） |

映射依据：本机唯一可用 provider 为 `builtin:zai-coding-plan`，其 `GLM-5.3` 注册的推理档位恰为
`low`/`high`/`max`（本机模型注册表与会话日志双重确认），因此采用 MODEL-MAPPING.md 的
Option B（同一模型、三档思考等级）——这是唯一完全被本机证据支撑的分层方案。
`glm-5-turbo`/`GLM-5.2` 等备选见 MODEL-MAPPING.md 备注及其前置验证条件。

## Files

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
- 未改动：`.zcode-plugin/plugin.json`、`commands/ultracode.md`、
  LICENSE、NOTICE.md、docs/01–07（结构本已合规；F-05 独立复核确认）

## Acceptance

- Static checks：S-01–S-09 全部 PASS（自动化脚本，一次性执行未入库）；S-10 PENDING（需 UI 安装后新开 session）。
- Functional scenarios：F-01–F-12 全部 PASS（12/12，详见 docs/08；必过项 F-01/F-02/F-05/F-07/F-09/F-12 全过）。
- Failed scenarios：无 FAIL。两点如实记录的局限：
  1. 功能场景中的 worker 以"内置 subagent + worker 系统提示词全文"替身执行——插件 agent 仅在
     安装后的新会话中加载；路由决策、契约、波次、升级、证据核验均为真实执行，
     但 low/high/max 的实际分层效果待安装后确认（docs/08 §4 步骤 3）。
  2. F-08 场景 1 中 fast 替身完成了跨模块任务（同模型替身无思考档位差异），未触发升级；
     场景 2 以确定性 blocked 前置完整验证了升级链路。安装后 thoughtLevel 差异会强化该边界。

## Scope Guard

- Hooks added: no
- MCP added: no
- Runtime code added: no（纯 JSON/Markdown 包）
- Persistent state added: no（任务图仅存在于会话上下文）

## Remaining Manual Steps

1. 将本仓库推送到 GitHub（marketplace.json 位于仓库根目录，布局与 GitHub marketplace 兼容）。
2. Settings → Plugins → Discover → `+` → 输入 GitHub 仓库链接 → 安装并启用。
3. 新开 session，确认 skill/command/三个 worker 可见（补齐 S-10）。
4. 对三个 worker 各派发一个最小任务，确认模型与思考档位按映射生效。

## Risks

- `builtin:zai-coding-plan/GLM-5.3` 为本机限定映射；换机或换 provider 需按 MODEL-MAPPING.md 步骤重配。
- `model` 字段使用 provider 限定形式（日志中的规范形态）；若安装后 agent 无法调用模型，
  回退为裸 ID `GLM-5.3` 是文档中的第一调整项（docs/04 §4）。
- Skill 自动触发依赖 description 质量（当前 308 字符，前载触发词）；若触发不稳，
  按docs/04 §7 顺序先调 description，不加 Hook。
