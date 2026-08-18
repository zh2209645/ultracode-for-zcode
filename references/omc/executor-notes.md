# OMC Executor 参考摘要

来源：

https://github.com/Yeachan-Heo/oh-my-claudecode/blob/main/agents/executor.md

本 MVP 只提炼以下原则：

- Executor 应只完成分配范围内的实现任务。
- 优先最小可行改动，避免相邻重构和过度抽象。
- 非简单任务先读取周边代码，匹配现有模式。
- 修改后运行与范围相称的诊断、构建或测试。
- 连续失败时应把证据交回 orchestrator，而不是无限尝试。
- 返回内容应包含改动位置、验证结果和剩余风险。
- Worker 不应承担总体架构、总计划或最终用户答复。

本 MVP 不复制 OMC executor 的长提示、Claude 专用工具名、LSP/MCP 工具矩阵、`.omc` 状态约定或二级委派能力。
