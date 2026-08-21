# FlowVerse Copilot Agent 用户指南

[English](AI_AGENT_USER_GUIDE.md)

FlowVerse 的设计目标是在 VS Code 原生 Copilot Chat 中使用。请从 `@flowverse` 开始，或者写出清晰、无歧义的 FlowVerse 请求，让 VS Code 将其路由到对应参与者。你可以使用日常的 Jira、产品、架构、风险、交付和测试语言；内部能力名称和语义模型名称并不是用户必须掌握的词汇。

扩展使用 Copilot Chat 中选择的模型。它不提供独立 AI 模型，因此控制权仍属于 VS Code 和企业模型策略。

运行时流程的可视化说明请参阅 [FlowVerse Copilot Agent 架构图](architecture/flowverse-copilot-agent-architecture-zh_CN.html)。

## Agent 可以做什么

FlowVerse 参与者可以：

- 获取 Jira Issue，并将其保留为 Agile Delivery 上下文。
- 优化需求、验收条件、缺口和待决问题。
- 检查相关系统、模块、接口、消息、流程、依赖、风险和证据。
- 比较方案并解释权衡，而不会把建议当作已批准决策。
- 分析影响、关键路径、兼容性、测试覆盖、漂移和发布就绪度。
- 生成基于 Issue 的结构化表示、静态图表和实现任务草案。
- 基于已保留证据解释决策及其备选方案。
- 准备 Jira 和 Workspace 文件变更计划，但不直接应用。
- 仅在原生确认和修订版本检查通过后，应用当前已评审计划。

Fetch 被有意限制在较小范围：它只获取并规范化 Jira 证据。可选分析和专用视图只在明确请求时运行。

## Jira 目标与上下文

FlowVerse 按以下顺序解析唯一 Jira 目标：

1. 当前请求中写明的单个 Jira Key。
2. 当前 FlowVerse 对话中最近使用的 Issue。
3. 当前 VS Code 窗口中唯一可用的已保留 Issue。
4. 当以上规则无法确定唯一 Issue 时提出澄清问题。

当前请求中的 Key 具有最高优先级。多个显式候选项或相互冲突的操作元数据属于歧义情况；FlowVerse 会询问，而不是猜测。

完整 Jira 证据、生成 Artifact 和待处理计划仅在当前窗口有效。成功采集后的内容在 30 分钟内为 Fresh；30 分钟后变为 Stale，但仍可使用。任何读取操作都不会暗中访问 Jira。当数据新鲜度很重要时，应要求 FlowVerse 刷新 Issue 或获取最新 Jira 状态。

成功刷新会创建新的来源修订版本。已有可选分析仍会保留其原始修订版本，并标记为 Stale，直到重新运行。如果刷新失败，最后一份有效上下文会继续保留并标记为 Stale。

重启后，FlowVerse 可能只保留已知 Issue Key 和时间戳等导航元数据。这些元数据不能满足请求或视为已缓存上下文。请通过可见的 Fetch 或 Refresh 恢复可用内容。

## 工作台布局

推荐布局：

- **编辑器区域：** Domain Canvas 和原始 JSON。
- **右侧边栏：** 带有 `@flowverse` 的原生 Copilot Chat。
- **底部面板：** 用于技术诊断的 FlowVerse Output。

VS Code 允许移动这些视图。布局不会影响 Jira 目标解析。

## 从本地 Jira 模拟器开始

在无法访问真实 Jira 实例时，可使用该流程开发或验证 FlowVerse。

模拟器是可选的配套开发项目，不属于本扩展仓库。请配置其 Multi-root Workspace，使 `plugin` Workspace 文件夹指向本仓库。

### 前置条件

- Node.js 20 或更高版本。
- 与扩展 `^1.128.0` Engine 要求兼容的 VS Code。
- VS Code 中可用的 GitHub Copilot Chat。
- 已安装一次扩展依赖：

  ```bash
  npm ci
  ```

### 启动完整调试环境

1. 在 VS Code 中打开配套模拟器的 `flowverse-offline-debug.code-workspace`。
2. 打开 **Run and Debug**。
3. 选择 **Run FlowVerse plugin with local Jira**。
4. 按 `F5`。

该启动项会在 `http://127.0.0.1:4876` 启动模拟器，启动 Extension Host 和 Webview Watcher，执行初始增量构建，并使用仅限模拟器的凭据打开隔离的 Extension Development Host。它不会在每次启动时运行全量清理构建。

只有在明确需要先执行一次干净的完整开发构建时，才使用 **Build and run FlowVerse plugin with local Jira**。

## 启动已安装的生产版本

创建 VSIX：

```bash
npm ci
npm run build:prod
```

通过 **Extensions: Install from VSIX...** 安装 `dist/flowverse-vscode-preview.vsix`。然后：

1. 运行 **FlowVerse: Set Jira Base URL** 并输入 Jira 实例 URL。
2. 运行 **FlowVerse: Set Jira PAT**，在原生 Secret Input 中输入 PAT。
3. 运行 **Developer: Reload Window**。

不要把凭据写入环境文件、Settings JSON、提示词或文档。VSIX 自带架构 Knowledge Pack；除非有意配置覆盖，否则不需要仓库根目录中的架构文件。

## 主要聊天工作流

### 获取 Jira 上下文

打开 Copilot Chat 并输入：

```text
@flowverse fetch FACTORY-100
```

显式参与者形式是最可预测的起点。`Fetch Jira issue FACTORY-100 with FlowVerse` 这样的普通请求在无歧义时也可以被识别。

首次成功 Fetch 时，Chat 会按顺序报告以下用户可理解阶段：

1. 连接 Jira。
2. 获取 Issue。
3. 解析已获取字段。
4. 处理验收条件。
5. 保留规范化后的来源上下文。

结果是已保留的 Agile Delivery 上下文、一段精简 Chat 摘要，以及聚焦到所请求 Work Item 的 Agile Delivery 画布。默认情况下，FlowVerse 最多采集三层层级和 75 个后代。重复 Fetch 会把其他根节点组合到同一个会话视图中；重叠的后代仍保留为一条共享记录。

可在不修改 Jira 或 Workspace 文件的情况下移除已获取的会话上下文：

```text
@flowverse clear fetched FACTORY-100
@flowverse clear all fetched delivery work
```

清除一个根节点时，仍被其他已获取层级引用的记录会继续保留。Workspace 中的 `*.jira.json` 记录始终不受影响。

### 从已保留上下文继续

在同一个 FlowVerse 对话中继续提出类似请求：

```text
@flowverse Create integration and end-to-end test scenarios for this issue.
```

只要该 Issue 仍是当前对话目标，就无需重复 Key。FlowVerse 会报告其保留证据为 Fresh 还是 Stale，然后运行所请求的只读工作，而不会暗中刷新。成功的命名结果只会更新对应的生成分析。

### 按用户结果划分的能力

请用自然语言描述需要的结果：

| 结果 | 请求示例 |
| --- | --- |
| 刷新 Jira 证据 | `Get the latest Jira state for this issue.` |
| 优化需求 | `Refine the scope, acceptance criteria, gaps, and open questions.` |
| 分析架构影响 | `Trace the affected systems, interfaces, flows, and owners.` |
| 比较方案 | `Compare two production-ready options and their tradeoffs.` |
| 规划交付 | `Draft implementation, verification, rollout, and rollback work.` |
| 分析阻塞项和关键路径 | `Separate confirmed blockers from risks and explain the safest order.` |
| 评估兼容性 | `Assess the one-release compatibility strategy and affected consumers.` |
| 生成测试场景 | `Create positive, negative, integration, validation, and end-to-end tests.` |
| 分析漂移 | `Compare approved intent with current architecture and workspace evidence.` |
| 评估发布就绪度 | `Explain each passed, review, failed, or unknown readiness gate.` |
| 生成结构化表示 | `Generate an issue-grounded structured representation.` |
| 生成交付图表 | `Generate an issue relationship diagram from retained context.` |
| 起草实现任务 | `Draft development, documentation, and test tasks without creating them.` |
| 解释决策 | `Explain why this option was selected and what alternatives were rejected.` |
| 检查架构 | `Inspect the relevant architecture evidence without opening a view.` |
| 打开专用图表 | `Open the layered architecture and focus the affected machine-telemetry adapter.` |

只读能力可以使用已保留证据并更新其自身生成分析。它们不会修改 Jira 或 Workspace 文件。

## 已保留上下文契约

Jira 采集会在当前 VS Code 窗口中保存一个经过规范化、由来源支撑的修订版本。FlowVerse 通过 Agile Delivery Canvas、FlowVerse Details 和明确请求的分析展示这些证据。

内容会明确标注为 **Jira Source**、**AI Generated**、**Tool Generated** 或 **User Note**。生成分析会保留生成时间、来源修订版本、新鲜度和 Provenance，而不暴露内部标识符。原始 Jira 证据在结构上保持真实，不会把生成文本改写成来源内容。

每种生成分析只激活最新成功版本。旧版本在当前窗口中仍可通过 Chat 查询。失败或取消的重新运行不会替换最新成功版本。外部提供的 User Note 会保留 Provenance，且在此工作流中不可修改。

生成结果按分析类型非破坏性更新。修改来源记录或用户创作内容必须使用已确认变更流程。

## 可见 UI 操作路由

FlowVerse 管理的按钮、后续操作、关联 Issue 链接和保留命令不会调用隐藏工作流。每个操作都会在焦点变化前捕获 Issue 和 Artifact 绑定，打开或显示 Chat，展示完整、可读的 `@flowverse` 提示词，并自动提交。

对于窗口中已保留的关联 Issue，可见提示词会聚焦对应 Work Item；对于未缓存的关联 Issue，则要求获取精确 Key。被点击的 Key 会包含在提示词中；选择链接不会改变隐藏的全局 Issue。

滚动、展开内容、复制文本、关闭标签页、拆分编辑器和原生编辑器控件等纯呈现交互仍在本地执行。

## 架构与可视化

架构检查为只读操作，不会打开视图。当需要 Layered Diagram 或其他专用可视化时，应显式要求打开、显示、可视化或交互式检查。普通分析不会自动打开图表。

选择图表元素只会改变选择状态，不会采纳建议、批准写入，也不会改变当前 Jira 目标。

## 安全地规划和应用变更

写入前先请求预览，例如：

```text
@flowverse Prepare exact Jira changes and a workspace file update for this issue. Show the complete plan and do not apply it.
```

规划为非破坏性操作，不需要语义确认。预览包含精确 Jira 操作和 Workspace 文件创建/覆盖操作、当前绑定、过期时间、风险、Plan ID 和单次 Digest。Workspace 目标必须位于当前 Workspace 文件夹内；路径穿越和 Workspace 外路径会被拒绝。

确认计划正确后，请求：

```text
@flowverse Apply the reviewed plan.
```

FlowVerse 随后通过 VS Code 原生确认展示完整预览，只有在你批准后才会应用。应用前会立即重新检查绑定到计划中的 Issue、上下文、架构、Jira 和文件内容修订版本。过期、已使用、已拒绝、不匹配或已过时的计划均不能应用。部分成功结果会标识每项成功和失败操作，变更不会自动重试。

该确认边界适用于 Jira 创建和更新、链接、状态转换、Workspace 文件创建或覆盖、用户创作内容变更和其他外部写入。Jira 读取、显式刷新、上下文读取、分析、生成分析 Upsert、显式可视化和计划预览不需要语义确认。

模拟器中的变更只存在于内存，在模拟器重启或重置后消失。

## 单版本兼容期

旧版 Jira Alias、原通用分析 Alias 和四个保留 Launch Command 只在包含此次重新设计的首个版本中注册。它们是弃用适配器，不参与正常 FlowVerse 能力选择，也不应在新工作流中使用。

在该版本中，它们保留现有公共输入和结果字段，同时通过与标准能力相同的上下文、刷新、可视化和确认边界。保留命令会显示并自动提交对应 Chat 提示词，而不会直接运行分析或打开视图。不受支持的旧版请求会被拒绝，不会恢复隐藏刷新、自动打开视图或较弱的写入确认。

## 手动 Copilot 冒烟测试

先对模拟器运行以下步骤，再使用已安装生产版本对有权限的真实 Jira 项目重复相关只读步骤：

1. 输入清晰无歧义的普通 FlowVerse 请求，确认 VS Code 正确路由，且不会拦截无关提示词。
2. 获取一个已知 Issue，观察五个有序进度阶段。
3. 确认 Issue 已保留，且没有打开报告或可选图表。
4. 在不重复 Key 的情况下请求分析。
5. 确认只有对应的生成分析发生变化，并显示来源、修订版本和新鲜度。
6. 选择已缓存和未缓存的关联 Issue，确认每个自动提交的可见提示词都包含被点击的 Key。
7. 调用每个保留 Launch Command，确认它会显示并提交 Chat 提示词，而不是运行隐藏工作流。
8. 要求检查架构，确认不会打开视图；然后显式要求打开图表，并确认只在此时打开。
9. 规划一个 Jira 变更和一个 Workspace 文件创建或覆盖操作。检查并拒绝一次预览；创建新计划并批准，确认修订版本和 Digest 强制生效。
10. 重启 Extension Development Host，确认此前完整 Issue 上下文不可使用，直到可见 Fetch 或 Refresh 成功。

真实 Copilot 验证应使用你有权读取的 Issue；除非有一次性测试项目，否则避免变更操作；并确认提示词、输出或视图均不暴露凭据或原始认证信息。

## 开发期间的 Watch 与 Reload

本地 Jira 启动项会启动持续运行的 Extension 和 Webview Watcher。

- 修改 Extension Host 后，等待 TypeScript Watcher 报告无错误，再在 Extension Development Host 中运行 **Developer: Reload Window**。
- 仅修改 Webview 后，等待 Vite 完成，再运行 **Developer: Reload Webview**。
- 修改 Manifest、共享协议或跨界面内容后，运行 **Developer: Reload Window**。
- 需要停止模拟器和 Watcher 时，使用 **Tasks: Terminate All Tasks**。

## 故障排查

### Chat 无法获取 Jira

1. 运行 **FlowVerse: Show Output**。
2. 本地开发时，确认 `curl http://127.0.0.1:4876/health` 成功。
3. 确认 Extension Development Host 从 `flowverse-offline-debug.code-workspace` 启动。
4. 生产环境中，运行 **FlowVerse: Set Jira Base URL** 和 **FlowVerse: Set Jira PAT**，然后运行 **Developer: Reload Window**。
5. 确认 `flowverse.jiraBaseUrl` 机器设置或 VS Code 进程环境中存在 Jira Base URL。

### 已保留上下文为 Stale

Stale 上下文仍可读取，且不会自行刷新。要求 FlowVerse 刷新 Issue 或获取最新 Jira 状态。如果刷新失败，仅在可以接受过时证据时继续使用保留的最后有效上下文。

### 图表没有打开

普通分析有意不打开专用视图。请显式要求打开或可视化目标图表。

### 兼容性或就绪度为 `unknown`

如果缺少所需的前后契约、测试、Jira 字段或 Workspace 证据，这是预期结果。补充或引用缺失证据后重新运行能力。FlowVerse 不会根据缺失证据制造高置信度结论。

### 计划无法应用

如果计划已过期、已使用、已拒绝，或任何绑定的 Jira、上下文、架构或文件修订版本发生变化，请创建新的预览。变更不会自动重试。
