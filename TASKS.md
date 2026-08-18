# MVP 实施任务清单

> 完成日期：2026-08-18。结果细节见 `docs/08-TEST-RECORD.md` 与 `docs/09-FINAL-REPORT.md`。

## A. 环境与模型

- [x] 确认当前 ZCode 版本支持插件内 Skill、Command 和 Subagent（官方文档 + 本机 grill 插件 agents 实际加载证据）
- [x] 确认本机已连接的模型列表（`~/.zcode/v2/config.json` provider 注册表）
- [x] 记录 fast、standard、deep 和 review 四个 Agent 的模型映射（`builtin:zai-coding-plan/GLM-5.3`，Option B 分层）
- [x] 若仅使用一个模型，确认其支持的 `thoughtLevel` 值（`low` / `high` / `max`）
- [x] 配置三个写 Agent 和一个只读 reviewer 的真实模型 ID

## B. 插件结构

> 离线静态校验全部通过（docs/08 §1）；2026-08-19 的应用内安装、`/ultracode` 和 worker-fast 证据仅适用于 0.1.0。0.1.3 尚需发布 tag、刷新 marketplace 并在新会话复验。插件 agent 不出现在 Settings → Subagents 属预期（该页仅列用户级 agent）。

- [x] 验证 `.zcode-plugin/plugin.json` 可被加载（schema 校验 + 与已安装插件对照）
- [x] 验证 `marketplace.json` 格式符合官方 marketplace schema，`git-subdir` source 结构可解析（当前 `v0.1.3` 远端解析待 S-12）
- [ ] 在 0.1.3 新会话验证 Skill 出现在 ZCode Skills 列表（结构静态合规；0.1.0 有历史证据）
- [ ] 在 0.1.3 新会话验证 `/ultracode` 出现在 Commands 列表（结构静态合规；0.1.0 有历史证据）
- [ ] 验证 fast/standard/deep/review 四个 Agent 出现在 Plugin subagents 列表（前三个已有历史证据；worker-review 需在 0.1.3 新 session 实测）

## C. 动态委派策略（F-01 至 F-12 为 0.1.0 历史证据，见 docs/08 §2）

- [x] 主 Agent 能识别“无需委派”的简单任务（F-01）
- [x] 主 Agent 能用内置 Explore 处理只读探索（F-02）
- [x] 主 Agent 在 0.1.0 历史场景中能按难度选择 fast / standard / deep（F-03/F-04/F-05/F-09；当前只读复核路由已由 reviewer 取代）
- [x] 主 Agent 能识别独立任务并并行派发（F-03/F-06，并发 ≤3）
- [x] 主 Agent 能识别依赖并按波次派发（F-07）
- [x] 主 Agent 能在 worker 失败后升级一个层级（F-08：blocked → 携证据单次升级）
- [x] 主 Agent 不会让 subagent 接管总任务规划（F-12）
- [x] 主 Agent 能汇总结果并执行最终验证（F-10 独立复跑证据）

## D. 验收

- [x] 完成 0.1.3 配置、权限、结构和校验和静态检查
- [ ] 完成 S-03/S-04/S-05/S-06/S-10 的 0.1.3 新会话发现、调用和加载验证
- [ ] 完成 S-12 的受保护或签名 tag 发布与远端安装验证
- [x] 完成 F-01–F-12 的 0.1.0 历史功能场景（12/12）
- [ ] 在 0.1.3 新会话重跑强制回归 F-01/F-02/F-05/F-07/F-09/F-13/F-14；历史 deep 只读复核证据不能替代当前 reviewer 路由验证
- [x] 记录实际委派轨迹（docs/08）
- [x] 确认没有新增 Hook
- [x] 确认没有新增 MCP
- [x] 确认没有新增 Node/TS/Python runtime
- [x] 确认没有新增持久状态目录
- [x] 检查 MIT 许可证和致谢（LICENSE、references/omc/LICENSE、NOTICE.md）
- [x] 生成最终实施报告（docs/09，待上述 PARTIAL 项完成后更新发布结论）
