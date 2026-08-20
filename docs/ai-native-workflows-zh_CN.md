# AI 原生工作流

[English](ai-native-workflows.md)

FlowVerse 让 LLM 和 Agent 使用与资源管理器、画布和详细信息工作台相同的来源支撑上下文和边界明确的语义命令。因此，Agent 工作流可以在产品模型上导航和执行操作，而不会创建一份 AI 专用的平行数据副本。

## LLM-command 模式

命令边界将模型推理与平台执行分开：

- FlowVerse 从当前由来源支撑的会话中解析实体和关系。
- 人类可读的请求映射为边界明确的导航、分析和变更命令。
- 扩展通过 UI 使用的同一组应用服务执行这些命令。
- 变更必须经过预览和确认，而不是直接应用模型输出。

## Copilot 原生 Launch Intelligence

`@flowverse` 使用 VS Code 中选择的模型。它不会提供额外的模型服务，也不会绕过企业模型策略。

可从以下请求开始：

```text
@flowverse fetch FACTORY-122
```

FlowVerse 可以采集 Jira 证据、打开结构化 Issue 报告、检查架构影响、比较方案、规划交付、分析阻塞项、评估兼容性与就绪度、生成测试或图表，并导航到相关画布。需要 Agent 处理的 UI 操作会转换为可见、可读的提示词，并通过同一参与者工作流提交。

只读分析和导航不需要语义确认。Jira 变更，以及 Workspace 文件创建或覆盖，会先生成预览和绑定修订版本的单次计划，并在应用前立即请求原生确认。

完整 Issue 上下文和生成的报告区块保留在当前 VS Code 窗口中。重新打开已保留报告不会暗中获取 Jira 数据；当服务器当前状态很重要时，应显式请求刷新。

获取一个 Work Item 时，会采集其有界后代层级（默认三层、最多 75 个后代），与 Workspace 中已有的来源记录合并，打开 Agile Delivery 画布，并聚焦所请求的条目。后续 Fetch 会把独立根节点组合到同一个内存会话中。清除一个根节点或全部已获取工作时，只移除会话采集的记录；Workspace 文件仍是权威来源且不会被修改。

配置、审批、能力验证和故障排查详见 [AI Agent 用户指南](AI_AGENT_USER_GUIDE-zh_CN.md)。画布操作所使用的共享命令边界详见[图表操作框架](architecture/diagram-action-framework-zh_CN.md)。
