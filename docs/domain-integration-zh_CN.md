# 领域集成

[English](domain-integration.md)

本指南介绍一个领域如何参与 FlowVerse、如何解析源记录，以及领域专属行为应放置在哪里。

## 领域扩展模型

每个受支持领域都是 `src/domains/` 下的一个包，并注册两个部分：

- Extension Host 模块，包含连接器和验证适配器；
- Renderer 模块，包含画布呈现、资源管理器行为、详细信息和视觉主题。

`AbstractDomainConnector` 管理共享初始化流程：解析领域 Worldview、将文件访问限制在 Workspace 边界内、解析文件、调用验证，并把验证后的注册表交给领域连接器。连接器把领域记录标准化为中立 Bridge Model；验证适配器负责验证领域 Worldview，并提取已注册的实例 ID。

通用组合加载器、资源管理器、画布、图例和详细信息外壳不包含 Agile Delivery 或 System Architecture 的特殊分支。新领域只需实现契约，并注册到 Extension 和 Renderer Registry。

当前领域包：

```text
src/domains/
  systemArchitecture/
    extension/   连接器和验证适配器
    canvas/      投影、布局、卡片、边和主题
    details/     架构详细信息模块
  agileDelivery/
    extension/   Jira 连接器、验证适配器和运行时来源
    canvas/      工作项呈现和主题
    details/     Agile Delivery 详细信息模块
    model/       标准 Jira 运行时合并模型
```

## 资源

仓库将产品知识与可丢弃的开发数据分开：

```text
resources/
  knowledge/     打包的定义、Schema 和运行时 Agile Delivery 配置
  examples/
    unified-worldview.instance.json
    architecture/
    agile-delivery/
    cross-domain-links/
```

`resources/knowledge/` 包含共享定义、Schema，以及供已连接会话来源使用的不含数据的 Agile Delivery 配置。扩展自有的 Unified Worldview 定义在 `unifiedWorldviewModule.ts` 中，不再从 Workspace 数据中发现。所有演示系统、模块、Jira 工作项和跨领域断言均位于 `resources/examples/` 下；在生产打包前可以移除它们，而不会改变产品运行时。

开发示例包含当前两个领域、23 个架构边界、24 条多级 Agile Delivery 记录，以及显式的交付到架构 Link Set。详见[智能工厂运营示例](smart-factory-operations-example-zh_CN.md)。

## 配置与数据解析

通用画布从扩展自有的双领域组合启动。System Architecture 从 Workspace 中的 `architecture/worldview.instance.json` 解析；Agile Delivery 记录与跨领域关系由已连接来源提供，并且只保留在当前运行时/会话组合中。

Jira Base URL 解析顺序：

1. VS Code 进程中的 `FLOWVERSE_JIRA_BASE_URL`；
2. 机器本地的 `flowverse.jiraBaseUrl` 设置；
3. 仅在 Extension Development Host 中使用 `dev.env`。

已安装的扩展应使用 **FlowVerse: Set Jira Base URL** 进行配置。Jira 凭据来自 VS Code SecretStorage 或进程变量 `JIRA_PAT`；不要把凭据写入环境文件或 Workspace 文件。
