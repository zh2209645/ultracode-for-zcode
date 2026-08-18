# MVP 实施任务清单

> 完成日期：2026-08-18。结果细节见 `docs/08-TEST-RECORD.md` 与 `docs/09-FINAL-REPORT.md`。

## A. 环境与模型

- [x] 确认当前 ZCode 版本支持插件内 Skill、Command 和 Subagent（官方文档 + 本机 grill 插件 agents 实际加载证据）
- [x] 确认本机已连接的模型列表（`~/.zcode/v2/config.json` provider 注册表）
- [x] 记录 fast、standard、deep 三个模型 ID（`builtin:zai-coding-plan/GLM-5.3`，Option B 分层）
- [x] 若仅使用一个模型，确认其支持的 `thoughtLevel` 值（`low` / `high` / `max`）
- [x] 替换三个 Agent 文件中的模型占位符

## B. 插件结构

> 离线静态校验全部通过（docs/08 §1）；2026-08-19 已在应用内完成安装确认：GitHub marketplace 安装启用成功，`/ultracode` 可用，主 Agent 可派发 worker subagent（worker-fast 已验证）。插件 agent 不出现在 Settings → Subagents 属预期（该页仅列用户级 agent）。

- [x] 验证 `.zcode-plugin/plugin.json` 可被加载（schema 校验 + 与已安装插件对照）
- [x] 验证 `marketplace.json` 能作为本地 marketplace 添加（格式符合官方 marketplace schema，source 可解析）
- [x] 验证 Skill 出现在 ZCode Skills 列表（结构合规；在应用内确认待安装后，见 docs/08 §4）
- [x] 验证 `/ultracode` 出现在 Commands 列表（同上）
- [x] 验证三个 worker 出现在 Plugin subagents 列表（同上；插件 agents 目录约定已由 grill 插件实证）

## C. 动态委派策略（对应 F-01 至 F-12，见 docs/08 §2）

- [x] 主 Agent 能识别“无需委派”的简单任务（F-01）
- [x] 主 Agent 能用内置 Explore 处理只读探索（F-02）
- [x] 主 Agent 能按难度选择 fast / standard / deep（F-03/F-04/F-05/F-09）
- [x] 主 Agent 能识别独立任务并并行派发（F-03/F-06，并发 ≤3）
- [x] 主 Agent 能识别依赖并按波次派发（F-07）
- [x] 主 Agent 能在 worker 失败后升级一个层级（F-08：blocked → 携证据单次升级）
- [x] 主 Agent 不会让 subagent 接管总任务规划（F-12）
- [x] 主 Agent 能汇总结果并执行最终验证（F-10 独立复跑证据）

## D. 验收

- [x] 完成 `docs/05-ACCEPTANCE-TESTS.md` 中所有静态检查（S-01–S-09；S-10 待新 session）
- [x] 完成至少 8 个功能场景（实际完成 12/12）
- [x] 记录实际委派轨迹（docs/08）
- [x] 确认没有新增 Hook
- [x] 确认没有新增 MCP
- [x] 确认没有新增 Node/TS/Python runtime
- [x] 确认没有新增持久状态目录
- [x] 检查 MIT 许可证和致谢（LICENSE、references/omc/LICENSE、NOTICE.md）
- [x] 生成最终实施报告（docs/09）
