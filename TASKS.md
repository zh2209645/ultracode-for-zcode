# MVP 实施任务清单

## A. 环境与模型

- [ ] 确认当前 ZCode 版本支持插件内 Skill、Command 和 Subagent
- [ ] 确认本机已连接的模型列表
- [ ] 记录 fast、standard、deep 三个模型 ID
- [ ] 若仅使用一个模型，确认其支持的 `thoughtLevel` 值
- [ ] 替换三个 Agent 文件中的模型占位符

## B. 插件结构

- [ ] 验证 `.zcode-plugin/plugin.json` 可被加载
- [ ] 验证 `marketplace.json` 能作为本地 marketplace 添加
- [ ] 验证 Skill 出现在 ZCode Skills 列表
- [ ] 验证 `/ultracode` 出现在 Commands 列表
- [ ] 验证三个 worker 出现在 Plugin subagents 列表

## C. 动态委派策略

- [ ] 主 Agent 能识别“无需委派”的简单任务
- [ ] 主 Agent 能用内置 Explore 处理只读探索
- [ ] 主 Agent 能按难度选择 fast / standard / deep
- [ ] 主 Agent 能识别独立任务并并行派发
- [ ] 主 Agent 能识别依赖并按波次派发
- [ ] 主 Agent 能在 worker 失败后升级一个层级
- [ ] 主 Agent 不会让 subagent 接管总任务规划
- [ ] 主 Agent 能汇总结果并执行最终验证

## D. 验收

- [ ] 完成 `docs/05-ACCEPTANCE-TESTS.md` 中所有静态检查
- [ ] 完成至少 8 个功能场景
- [ ] 记录实际委派轨迹
- [ ] 确认没有新增 Hook
- [ ] 确认没有新增 MCP
- [ ] 确认没有新增 Node/TS/Python runtime
- [ ] 确认没有新增持久状态目录
- [ ] 检查 MIT 许可证和致谢
- [ ] 生成最终实施报告
