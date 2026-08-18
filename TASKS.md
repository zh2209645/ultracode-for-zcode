# MVP 实施任务清单

> 完成日期：2026-08-18；0.1.3 复验更新：2026-08-19。结果细节见 `docs/08-TEST-RECORD.md` 与 `docs/09-FINAL-REPORT.md`。
>
> English status note: the v0.1.3 plugin (updated in-app via GitHub) passed all mandatory
> regression scenarios F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14 (8/8) plus static checks
> S-03/S-04/S-05/S-06/S-10 on ZCode 3.7.7 (2026-08-19). S-12 was closed by the GPG-signed
> v0.1.4 release. Full evidence: docs/08 §11–§12, docs/09.

## A. 环境与模型

- [x] 确认当前 ZCode 版本支持插件内 Skill、Command 和 Subagent（官方文档 + 本机 grill 插件 agents 实际加载证据）
- [x] 确认本机已连接的模型列表（`~/.zcode/v2/config.json` provider 注册表）
- [x] 记录 fast、standard、deep 和 review 四个 Agent 的模型映射（`builtin:zai-coding-plan/GLM-5.3`，Option B 分层）
- [x] 若仅使用一个模型，确认其支持的 `thoughtLevel` 值（`low` / `high` / `max`）
- [x] 配置三个写 Agent 和一个只读 reviewer 的真实模型 ID

## B. 插件结构

> 离线静态校验全部通过（docs/08 §1）；0.1.3 已于 2026-08-19 经 GitHub marketplace 应用内更新安装，并在新会话完成 Skill/Command/四类 Agent 的加载与实际派发复验（docs/08 §11）。插件 agent 不出现在 Settings → Subagents 属预期（该页仅列用户级 agent），以实际派发记录为准。

- [x] 验证 `.zcode-plugin/plugin.json` 可被加载（schema 校验 + 与已安装插件对照）
- [x] 验证 `marketplace.json` 格式符合官方 marketplace schema，`git-subdir` source 结构可解析（`v0.1.3` 远端 tag 已发布并经 `ls-remote` 验证指向发布提交；S-12 仅剩"受保护或签名"加固）
- [x] 在 0.1.3 新会话验证 Skill 加载（2026-08-19：Skill 从 0.1.3 缓存路径实际注入会话，docs/08 S-03）
- [x] 在 0.1.3 新会话验证 `/ultracode` 可调用（2026-08-19 实测，docs/08 S-04）
- [x] 验证 fast/standard/deep/review 四个 Agent 可实际派发（2026-08-19：standard×2、deep×2、fast×2、review×3，docs/08 S-05/S-06）

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
- [x] 完成 S-03/S-04/S-05/S-06/S-10 的 0.1.3 新会话发现、调用和加载验证（2026-08-19，ZCode 3.7.7）
- [x] 完成 S-12 的受保护或签名 tag 发布与远端安装验证（0.1.4 收口：`git tag -s` 签名发布、marketplace ref 固定 `v0.1.4`、三处版本一致；GitHub 应用内更新链路已于 0.1.3 实证，docs/08 §12）
- [x] 完成 F-01–F-12 的 0.1.0 历史功能场景（12/12）
- [x] 在 0.1.3 新会话重跑强制回归 F-01/F-02/F-05/F-07/F-09/F-12/F-13/F-14（8/8 PASS，2026-08-19，docs/08 §11）
- [x] 记录实际委派轨迹（docs/08）
- [x] 确认没有新增 Hook
- [x] 确认没有新增 MCP
- [x] 确认没有新增 Node/TS/Python runtime
- [x] 确认没有新增持久状态目录
- [x] 检查 MIT 许可证和致谢（LICENSE、references/omc/LICENSE、NOTICE.md）
- [x] 生成最终实施报告（docs/09；0.1.3 回归结论已更新，仅剩 S-12 tag 加固结论待发布后更新）
