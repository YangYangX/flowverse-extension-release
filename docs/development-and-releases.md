# Development and Releases

This guide covers the local development loop, validation, production packaging, automated releases, and repository layout.

## Development

Requirements:

- Node.js 20 or newer;
- VS Code 1.128 or newer;
- npm.

Install and build:

```bash
npm ci
npm run build
```

Open the repository folder in VS Code, select **Run FlowVerse with Smart Factory Example**, and press `F5`. The Extension Development Host opens `resources/examples/`, allowing the System Architecture, Agile Delivery, and Unified Worldview canvases to use the same development dataset. Select **Run FlowVerse** instead to start with an empty workspace.

Useful commands:

- **FlowVerse: Open Unified Worldview Canvas**
- **FlowVerse: Open System Architecture Canvas**
- **FlowVerse: Open Agile Delivery Canvas**

The extension host is compiled with TypeScript and the webview with Vite. There is no esbuild step.

## Validation

Run the local checks with:

```bash
npm test
npm run typecheck:extension
npm run typecheck:webview
npm run build
```

The test suite covers model and schema boundaries, domain connectors, source precedence, cross-domain edge authority, generic canvas behavior, hierarchy layout, explorer focus, details tables, minimap geometry, Jira normalization, and Launch Intelligence.

## Production Package

Create an installable VSIX with:

```bash
npm ci
npm run build:prod
```

The package is written to:

```text
dist/flowverse-vscode-preview.vsix
```

Production packaging includes bundled knowledge and a credential-free `prod.env` reserved for non-secret runtime defaults, while excluding examples, `dev.env`, source files, tests, and development documentation. Jira connection settings are supplied by the installed VS Code host.

## Continuous Integration and Releases

Every push to `master` runs the tests, both type checks, extension-host tests, and a production VSIX build. The VSIX is retained as a GitHub Actions artifact for that commit.

To publish a release, include the exact uppercase token `RELEASE` in the latest Conventional Commit message. Commits since the previous `v*` release determine the next version: breaking changes increment major, `feat` increments minor, and other changes increment patch. The workflow updates the package version, commits it as `chore: publish vX.Y.Z [skip ci]`, creates the matching tag and GitHub Release, and attaches `flowverse-vscode-vX.Y.Z.vsix` with commit-based release notes.

After creating the release, the workflow uses the `FLOWVERSE_RELEASE_REPO_TOKEN` secret to update `YangYangX/flowverse-extension-release`. It mirrors `README.md`, `README-zh_CN.md`, `docs/`, and the complete Smart Factory dataset at `resources/examples/`, removes stale mirrored content, and publishes the current VSIX at `releases/flowverse-vscode-vX.Y.Z.vsix`.

## Repository Structure

```text
src/
  domains/      Domain-owned connectors, validation, presentation, and details
  extension/    VS Code activation and feature adapters
  flowverse/    Generic canvas, design system, layout, and architecture primitives
  shared/       Cross-runtime bridge and webview protocols
  webview/      React applications and VS Code webview bindings
resources/
  knowledge/    Packaged domain and schema contracts
  examples/     Removable development data
docs/           Product, architecture, and workflow documentation
```

`src/extension/extension.ts` is the activation composition root. Domain behavior belongs in `src/domains/`; shared canvas behavior belongs in `src/flowverse/workbench/`; the composition bridge belongs in `src/extension/features/bridge/`.
