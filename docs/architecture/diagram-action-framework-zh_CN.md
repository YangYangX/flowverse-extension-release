# FlowVerse 图表操作框架

[English](diagram-action-framework.md)

架构画布通过一条共享操作路径进行控制：

1. 在 `src/extension/features/diagramActions/registry.ts` 中注册能力。
2. 通过 `DiagramReferenceResolver` 解析面向用户的标签或稳定 ID。
3. 通过 `DiagramActionExecutor` 执行。
4. 通过 LM Tool、VS Code 命令、资源管理器手势或聊天链接公开同一操作。

这样可以保持直接画布交互与 AI 辅助交互一致。聊天结果、命令面板操作和 Tool Call 应产生相同的状态更新和验证行为。

## 核心组成

- `DiagramActionRegistry` 存储稳定操作 ID、命令 ID、目标类型、类别、安全元数据、别名和示例。
- `DiagramActionContextProvider` 读取当前 Layered Canvas，并可在操作允许时打开默认 Unified Worldview。
- `DiagramReferenceResolver` 将自然语言目标映射为 Graph Reference。遇到歧义时返回可点击的命令参数，而不是猜测。
- `DiagramActionExecutor` 验证参数、检查可用性、解析目标，并调用 `LayeredController` API。
- `registerDiagramActionCommands` 公开 `flowverse.diagram.execute` 以及每个操作对应的命令，并使用带版本的参数。

## 公共入口

当前 LM Tool 保持稳定：

- `flowverse_open_diagram` 打开解析后的服务预览或 Unified Worldview。
- `flowverse_control_diagram` 接受现有 kebab-case 画布操作，并转发到共享 Executor。

可点击聊天操作应使用：

```json
{
  "version": 1,
  "actionId": "diagram.focusNode",
  "args": {
    "nodeId": "vendor-adapter-service"
  }
}
```

命令 ID 为 `flowverse.diagram.execute`。

## 添加操作

首先添加 Registry 定义，包括：

- 稳定的 `id` 和 `commandId`
- 可选的旧版 `toolAction`
- `category`
- 支持的 `targetTypes`
- `channels`
- 安全和可用性元数据
- 必需输入字段
- 示例和别名

然后在 `DiagramActionExecutor` 中实现该操作。优先调用 `LayeredController` 方法，而不是直接发送 Webview Message。

只改变视图状态的操作不需要语义确认或原生确认。如果操作会修改 Jira、文件或已保留上下文，则不应直接加入此框架；它需要独立的规划与确认路径。
