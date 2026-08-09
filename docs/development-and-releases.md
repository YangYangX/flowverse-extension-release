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

Open [flowverse-plugin.code-workspace](../flowverse-plugin.code-workspace), select **Run FlowVerse**, and press `F5`. The Extension Development Host opens `resources/examples/`, allowing the System Architecture, Agile Delivery, and Unified Worldview canvases to use the same development dataset.

Useful commands:

- **FlowVerse: Open Unified Worldview Canvas**
- **FlowVerse: Open System Architecture Canvas**
- **FlowVerse: Open Agile Delivery Canvas**
- **FlowVerse: Toggle JSON/Canvas View**

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

Production packaging includes `prod.env` and bundled knowledge, while excluding examples, `dev.env`, source files, tests, and development documentation.

## Continuous Integration and Releases

Every push to `master` runs the tests, both type checks, extension-host tests, and a production VSIX build. The VSIX is retained as a GitHub Actions artifact for that commit.

To publish a release, include the exact uppercase token `RELEASE` in the latest Conventional Commit message. Commits since the previous `v*` release determine the next version: breaking changes increment major, `feat` increments minor, and other changes increment patch. The workflow updates the package version, commits it as `chore: publish vX.Y.Z [skip ci]`, creates the matching tag and GitHub Release, and attaches `flowverse-vscode-vX.Y.Z.vsix` with commit-based release notes.

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
