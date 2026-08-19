# 09 — 最终实施报告

- 日期：2026-08-19
- Plugin version：0.1.5（签名 tag `v0.1.5` 发布；0.1.5 含 F-15 的强制回归已于发布当日在 0.1.5 新会话补测通过，见 docs/08 §14；更早的回归实测基于 0.1.3，0.1.4 为 docs-only 升级）
- ZCode 环境：Windows 桌面版（0.1.5 回归当日为 3.7.7；2026-08-19 升级 3.8.1，核心功能复验通过，见 docs/08 §15），激活 provider `builtin:zai-coding-plan`（Z.ai Coding Plan）

## Executive Summary (EN)

The v0.1.3 plugin was updated in-app from the GitHub marketplace and verified on ZCode 3.7.7
(2026-08-19). All eight mandatory regression scenarios passed with real plugin subagents:
F-01 (trivial fix done directly, no dispatch), F-02 (Explore-only call-chain evidence),
F-05 (Explore → worker-review pre-review → worker-deep refactor → primary-run tests →
worker-review post-review; 22/22 tests plus 3 semantic probes, both verdicts `pass`),
F-07 (schema → parser → tests in strict dependency waves; 27/27 tests), F-09 (worker-deep
authorization change, permission probes, independent reviewer `pass`), F-12 (primary-Agent
control, 10 atomic dispatches, no nested delegation), F-13 (injection-resistant write worker,
sentinel hash unchanged, zero out-of-scope I/O), and F-14 (read-only worker-review hard
isolation: Read/Glob/Grep only, injection refused, no `.zcode` access, full contract format).
Static checks S-03/S-04/S-05/S-06/S-10 passed in the same 0.1.3 session. S-12 was closed by
the docs-only v0.1.4 release, which pins the marketplace ref to the GPG-signed `v0.1.4` tag
(key 6B699BE4A10CE49F, `git verify-tag` good signature). Full log: docs/08 §11–§12.

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

## 0.1.4 Formal Release

- Release type：documentation and release-integrity update; Skill, Command, and Agent behavior is unchanged from the verified 0.1.3 package.
- Version alignment：`marketplace.json`, `.zcode-plugin/plugin.json`, and Skill metadata all report 0.1.4; marketplace source ref is `v0.1.4`.
- Release integrity：`v0.1.4` was published as a GPG-signed tag using ed25519 key `6B699BE4A10CE49F`; `git verify-tag v0.1.4` returned a good signature.
- Acceptance impact：S-12 moved from PARTIAL at the 0.1.3 checkpoint to PASS. Combined with the unchanged-runtime 0.1.3 regression, v0.1.4 has no remaining mandatory acceptance failures.
- Scope guard：no Hook, MCP server, executable runtime, or persistent workflow state was introduced.

## 0.1.5 Orchestrator Mode

- Release type: prompt behavior update. Skill and `/ultracode` now (1) size the whole request before acting — direct work only within a "reading or modifying a few files" scope, dynamic workflow planning beyond it; (2) keep the primary context to plans, decisions, and verified conclusions, with targeted path/line evidence instead of bulk file reads; (3) run `/ultracode` in orchestrator mode — execution delegated by default, with three bounded direct-execution exceptions: trivial work (typically a single localized edit), subagent explicit failure/incompleteness once re-dispatch or escalation is exhausted (worker-review adjudication first after two failures), and genuinely unsplittable interrelated multi-file work (sequential pipelines of self-contained steps explicitly excluded).
- docs/03 and docs/06 were rewritten in English; docs/05 gained the F-15 orchestrator-mode scenario, now a mandatory gate.
- Validation: three independent adversarial review rounds (round 3 included a blind full-changeset pass), all verdicts pass; 5 defects fixed in round 1, 3 in round 2, zero new in round 3. Static S-01..S-12 conditions preserved: declarative-only package, versions synchronized to 0.1.5, marketplace ref pinned to the GPG-signed `v0.1.5` tag, checksums regenerated.
- Open obligation: CLOSED (2026-08-19, later the same day). The F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14/F-15 live regression plus the three v0.1.5 fix-seam probes was run in a fresh 0.1.5 session and passed in full — 22 real plugin dispatches, routing accuracy 9/9, deep misuse 0, zero nested delegation, zero infinite retries; full record in docs/08 §14.
- Post-update re-verification (2026-08-19, ZCode 3.8.1): after the host update, a fresh `/ultracode` orchestrator session re-verified core functionality — plugin enabled, `/ultracode` + skill mounting, four-agent registration, live Explore ×2 / worker-fast write probe / worker-review verdict (pass) with read-only boundaries honored, cache==source before the doc update. Proportionate core re-verification; the §14 full behavioral regression was not rerun. Full record in docs/08 §15.
- Scope guard: no Hook, MCP server, executable runtime, or persistent workflow state was introduced.

## Acceptance

- Static checks：全部 PASS。0.1.3 配置、权限和结构检查 32/32 PASS；S-03/S-04/S-05/S-06/S-10 已于 2026-08-19 在 0.1.3 新会话实测通过（Skill 加载、`/ultracode` 可用、四类 agent 实际派发、模型/档位生效）；0.1.4 通过同步版本字段、固定 marketplace ref 并发布 GPG 签名 `v0.1.4` tag 关闭 S-12；0.1.5 会话补测 Skill 自 0.1.5 缓存路径正式挂载、四类 agent 于 0.1.5 继续实际派发（docs/08 §14）。无剩余静态门禁失败项。
- Functional scenarios：0.1.3 强制回归 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14 已于 2026-08-19 在 0.1.3 新会话全部实跑 PASS（真实插件 subagent，非替身；路由正确率 8/8，deep 滥用率 0，无 nested delegation、无无限重试），含 F-13/F-14 注入诱导与 symlink/junction/TOCTOU 路径攻击拦截；完整记录见 docs/08 §11。
- In-app confirmation（2026-08-19，ZCode 3.7.7）：插件经 GitHub marketplace（git-subdir）在应用内更新至 0.1.3；`/ultracode` 可用；worker-fast/worker-standard/worker-deep/worker-review 四类均实际派发成功（standard×2、deep×2、fast×2、review×3、Explore×1）。插件 agent 不出现在 Settings → Subagents 属预期（该页仅列用户级 agent）。
- 如实记录的局限：0.1.0 历史场景曾以"内置 subagent + worker 系统提示词全文"替身执行（插件安装前）；0.1.3 强制回归已全部改为真实插件 subagent。low/high/max 档位差异未做逐一量化对比（各 worker 一次通过，无升级样本）；`.zcode` todo/log 豁免未主动行使（观察为零写入的更严格情形）。

## Scope Guard

- Hooks added: no
- MCP added: no
- Runtime code added: no（纯 JSON/Markdown 包；worker 运行时仅有文件工具）
- Persistent state added: no（任务图仅存在于会话上下文；写文件任务可更新 ZCode 工作区 `.zcode` todo/log，但插件不建立恢复数据库）

## Remaining Manual Steps

- ~~Mandatory: re-run the F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14/F-15 regression on the released 0.1.5 package in a fresh ZCode session before the next release (include the three v0.1.5 fix-seam probes listed in docs/08 §13).~~ 已完成（2026-08-19，docs/08 §14）：九场景 + 三探针全部 PASS；22 次真实插件委派；探针(ii) 的裁决后接写分支未行使（裁决推荐 void 并被采纳，门本身已验证），`.zcode`/档位差异等局限如实记录于 §14。
- Optional：add GPG public key `6B699BE4A10CE49F` to GitHub (Settings → SSH and GPG keys) so the tag displays the Verified badge; refresh the ZCode marketplace to 0.1.5 and recheck loading in a new session.
- 已完成（2026-08-19，docs/08 §11）：0.1.3 新会话加载（S-03/04/05/06/10）与四类 agent 实际派发；强制回归 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14 实跑 PASS；写任务确认只更新 SCOPE 文件（本次未行使 `.zcode` todo/log 豁免，为零写入）；worker-review 确认只有 Read/Glob/Grep、不访问 `.zcode`、只通过 ZCode subagent 返回结果；writer/reviewer 的 symlink/junction/reparse-point 穿透与 TOCTOU 并发替换场景确认每次访问前重新解析、拒绝一切越出可信工作区/SCOPE 的访问，宿主不能原子绑定真实目标时任务判 blocked。

## Risks

- `builtin:zai-coding-plan/GLM-5.3` 为本机限定映射；换机或换 provider 需按 MODEL-MAPPING.md 步骤重配。
- `model` 字段使用 provider 限定形式（日志中的规范形态）；若安装后 agent 无法调用模型，
  回退为裸 ID `GLM-5.3` 是文档中的第一调整项（docs/04 §4）。
- Skill 自动触发依赖 description 质量（当前已前载触发词）；若触发不稳，
  按docs/04 §7 顺序先调 description，不加 Hook。
- `SCOPE`、`.zcode` 和链接路径门禁属于 prompt/编排契约而非路径 sandbox；canonical 检查与访问之间仍有 TOCTOU 风险，主 Agent必须在每次访问前重新验证规范化真实目标并防止并发替换，实际写目标仍须由 Confirm Before Changes 或等价宿主权限逐项批准；宿主不能原子绑定目标时任务 blocked。
