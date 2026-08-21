# 开发与发布

[English](development-and-releases.md)

本指南介绍本地开发循环、验证、生产打包、自动发布和仓库结构。

## 开发

环境要求：

- Node.js 20 或更高版本；
- VS Code 1.128 或更高版本；
- npm。

安装并构建：

```bash
npm ci
npm run build
```

在 VS Code 中打开仓库文件夹，选择 **Run FlowVerse with Smart Factory Example**，然后按 `F5`。Extension Development Host 会打开 `resources/examples/`，让 System Architecture、Agile Delivery 和 Unified Worldview 画布使用同一开发数据集。选择 **Run FlowVerse** 则会从空 Workspace 启动。

常用命令：

- **FlowVerse: Open Unified Worldview Canvas**
- **FlowVerse: Open System Architecture Canvas**
- **FlowVerse: Open Agile Delivery Canvas**

Extension Host 使用 TypeScript 编译，Webview 使用 Vite 构建，不包含 esbuild 步骤。

## 验证

运行本地检查：

```bash
npm test
npm run typecheck:extension
npm run typecheck:webview
npm run build
```

测试套件覆盖模型与 Schema 边界、领域连接器、来源优先级、跨领域边权威、通用画布行为、层级布局、资源管理器聚焦、详细信息表格、Minimap 几何、Jira 标准化和 Launch Intelligence。

## 生产包

创建可安装 VSIX：

```bash
npm ci
npm run build:prod
```

输出路径：

```text
dist/flowverse-vscode-preview.vsix
```

生产包包含内置知识和不含凭据的 `prod.env`，该文件仅保留给非敏感运行时默认值；示例、`dev.env`、源文件、测试和开发文档不会进入 VSIX。Jira 连接设置由安装扩展的 VS Code Host 提供。

## 持续集成与发布

每次推送到 `master` 都会运行测试、两项类型检查、Extension Host 测试和生产 VSIX 构建。该提交的 VSIX 会保留为 GitHub Actions Artifact。

如需发布版本，请在最新 Conventional Commit 消息中包含完全大写的 `RELEASE` 标记。自上一个 `v*` 版本以来的提交决定下一个版本号：Breaking Change 增加 Major，`feat` 增加 Minor，其他变更增加 Patch。工作流会更新包版本，将其提交为 `chore: publish vX.Y.Z [skip ci]`，创建对应 Tag 和 GitHub Release，并附带使用提交生成 Release Notes 的 `flowverse-vscode-vX.Y.Z.vsix`。

创建 Release 后，工作流使用 `FLOWVERSE_RELEASE_REPO_TOKEN` Secret 更新 `YangYangX/flowverse-extension-release`。它同步 `README.md`、`README-zh_CN.md`、`docs/` 和 `resources/examples/` 下的完整智能工厂数据集，移除过期的镜像内容，并将当前 VSIX 发布到 `releases/flowverse-vscode-vX.Y.Z.vsix`。

## 仓库结构

```text
src/
  domains/      领域管理的连接器、验证、呈现和详细信息
  extension/    VS Code 激活与功能适配器
  flowverse/    通用画布、设计系统、布局和架构基础组件
  shared/       跨运行时 Bridge 和 Webview 协议
  webview/      React 应用与 VS Code Webview 绑定
resources/
  knowledge/    打包的领域与 Schema 契约
  examples/     可移除的开发数据
docs/           产品、架构和工作流文档
```

`src/extension/extension.ts` 是激活组合根。领域行为属于 `src/domains/`；共享画布行为属于 `src/flowverse/workbench/`；组合 Bridge 属于 `src/extension/features/bridge/`。
