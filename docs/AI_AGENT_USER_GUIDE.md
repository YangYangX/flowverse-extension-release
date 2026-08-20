# FlowVerse Copilot Agent User Guide

FlowVerse is designed to be used through native Copilot Chat in VS Code. Start with `@flowverse`, or write a plain, unambiguous FlowVerse request and let VS Code route it to the participant. You can use ordinary Jira, product, architecture, risk, delivery, and testing language; internal capability and semantic-model names are not required user vocabulary.

The extension uses the model selected in Copilot Chat. It does not provide a separate AI model, so VS Code and enterprise model policies remain in control.

For a visual explanation of the runtime flow, open the [FlowVerse Copilot agent architecture diagram](architecture/flowverse-copilot-agent-architecture.html).

## What the Agent Does

The FlowVerse participant can:

- Fetch and retain a Jira issue as Agile Delivery context.
- Refine requirements, acceptance criteria, gaps, and open questions.
- Inspect relevant systems, modules, interfaces, messages, flows, dependencies, risks, and evidence.
- Compare solutions and explain tradeoffs without treating a proposal as an approved decision.
- Analyze impact, critical path, compatibility, test coverage, drift, and release readiness.
- Generate issue-grounded structured representations, static diagrams, and implementation-task drafts.
- Explain decisions and their alternatives from retained evidence.
- Prepare Jira and workspace-file change plans without applying them.
- Apply only a current reviewed plan after native confirmation and revision checks.

Fetching is deliberately narrow: it acquires and normalizes Jira evidence. Optional analyses and specialized views run only when requested.

## Jira Target and Context

FlowVerse resolves one Jira target in this order:

1. A single Jira key written in the current request.
2. The most recently used issue in the current FlowVerse conversation.
3. The sole usable issue retained in the current VS Code window.
4. A clarification question when none of those rules yields one issue.

A key in the current request takes precedence. Multiple explicit candidates or conflicting action metadata are ambiguous; FlowVerse asks instead of guessing.

Full Jira evidence, generated artifacts, and pending plans are window-scoped. A successful acquisition is fresh for 30 minutes. At 30 minutes it becomes stale but remains usable, and no read silently contacts Jira. Ask FlowVerse to refresh the issue or retrieve the latest Jira state when freshness matters.

A successful refresh creates a new source revision. Existing optional analyses retain their original revision and are labeled stale until you rerun them. If refresh fails, the last good context remains available and stale.

Across restart, FlowVerse may retain only navigation metadata such as known issue keys and timestamps. That metadata cannot satisfy a request or count as cached context. Use a visible fetch or refresh to restore usable content.

## Workbench Layout

The recommended layout is:

- **Editor area:** domain canvases, raw JSON, and service previews.
- **Right sidebar:** native Copilot Chat with `@flowverse`.
- **Bottom panel:** FlowVerse Output for technical diagnostics.

VS Code lets you move these views. The layout does not affect Jira target resolution.

## Start with the Local Jira Simulator

Use this flow when developing or verifying FlowVerse without access to a real Jira instance.

The simulator is an optional companion development project, not part of this extension repository. Configure its multi-root workspace so the `plugin` workspace folder points to this repository.

### Prerequisites

- Node.js 20 or newer.
- VS Code compatible with the extension's `^1.128.0` engine requirement.
- GitHub Copilot Chat access in VS Code.
- Plugin dependencies installed once:

  ```bash
  npm ci
  ```

### Launch the complete debug environment

1. Open the companion simulator's `flowverse-offline-debug.code-workspace` in VS Code.
2. Open **Run and Debug**.
3. Select **Run FlowVerse plugin with local Jira**.
4. Press **F5**.

This launch starts the simulator at `http://127.0.0.1:4876`, starts the extension-host and webview watchers, performs their initial incremental builds, and opens an isolated Extension Development Host with a simulator-only credential. It does not run a clean full build on every launch.

Use **Build and run FlowVerse plugin with local Jira** only when you explicitly need a clean full development build first.

## Start an Installed Production Build

Create the VSIX:

```bash
npm ci
npm run build:prod
```

Install `dist/flowverse-vscode-preview.vsix` with **Extensions: Install from VSIX...**. Then:

1. Run **FlowVerse: Set Jira Base URL** and enter the Jira instance URL.
2. Run **FlowVerse: Set Jira PAT** and enter the PAT into the native secret input.
3. Run **Developer: Reload Window**.

Do not place credentials in environment files, settings JSON, prompts, or documentation. The VSIX contains its own architecture knowledge pack; repository-root architecture files are not required unless you intentionally configure an override.

## Primary Chat Workflow

### Fetch Jira context

Open Copilot Chat and enter:

```text
@flowverse fetch FACTORY-100
```

The explicit participant form is the most predictable starting point. A plain request such as `Fetch Jira issue FACTORY-100 with FlowVerse` can also be detected when it is unambiguous.

During a successful first fetch, Chat reports these user-meaningful stages in order:

1. Connecting to Jira.
2. Retrieving the issue.
3. Parsing retrieved fields.
4. Processing acceptance criteria.
5. Retaining the normalized source context.

The result is retained Agile Delivery context, a concise Chat summary, and an Agile Delivery canvas focused on the requested work item. By default, FlowVerse acquires up to three hierarchy levels and 75 descendants. Repeated fetches compose additional roots into the same session view; overlapping descendants remain a single shared record.

Remove fetched session context without changing Jira or workspace files:

```text
@flowverse clear fetched FACTORY-100
@flowverse clear all fetched delivery work
```

Clearing one root preserves records still referenced by another fetched hierarchy. Workspace `*.jira.json` records remain available throughout.

### Continue from retained context

Continue in the same FlowVerse conversation and ask, for example:

```text
@flowverse Create integration and end-to-end test scenarios for this issue.
```

You do not need to repeat the key while it remains the conversation target. FlowVerse reports whether its retained evidence is fresh or stale, then runs the requested read-only work without refreshing implicitly. A successful named result updates only its corresponding generated analysis.

### Capabilities by user outcome

Ask in natural language for the outcome you need:

| Outcome | Example request |
| --- | --- |
| Refresh Jira evidence | `Get the latest Jira state for this issue.` |
| Refine requirements | `Refine the scope, acceptance criteria, gaps, and open questions.` |
| Analyze architecture impact | `Trace the affected systems, interfaces, flows, and owners.` |
| Compare solutions | `Compare two production-ready options and their tradeoffs.` |
| Plan delivery | `Draft implementation, verification, rollout, and rollback work.` |
| Analyze blockers and critical path | `Separate confirmed blockers from risks and explain the safest order.` |
| Assess compatibility | `Assess the one-release compatibility strategy and affected consumers.` |
| Generate test scenarios | `Create positive, negative, integration, validation, and end-to-end tests.` |
| Analyze drift | `Compare approved intent with current architecture and workspace evidence.` |
| Assess release readiness | `Explain each passed, review, failed, or unknown readiness gate.` |
| Generate a structured representation | `Generate an issue-grounded structured representation.` |
| Generate a delivery diagram | `Generate an issue relationship diagram from retained context.` |
| Draft implementation tasks | `Draft development, documentation, and test tasks without creating them.` |
| Explain a decision | `Explain why this option was selected and what alternatives were rejected.` |
| Inspect architecture | `Inspect the relevant architecture evidence without opening a view.` |
| Open a specialized diagram | `Open the layered architecture and focus the affected machine-telemetry adapter.` |

Read-only capabilities can use retained evidence and update their own generated analysis. They do not change Jira or workspace files.

## Retained Context Contract

Jira acquisition stores a normalized, source-backed revision for the current VS Code window. FlowVerse exposes that evidence through the Agile Delivery canvas, FlowVerse Details, and explicitly requested analyses.

Content is visibly attributed as **Jira Source**, **AI Generated**, **Tool Generated**, or **User Note**. Generated analyses preserve generation time, source revision, freshness, and provenance without exposing internal identifiers. Original Jira evidence remains structurally faithful and is not rewritten as if generated text were source.

Only the latest successful version of each generated analysis is active. Prior versions remain queryable through Chat for the current window. A failed or cancelled rerun does not replace the latest successful version. Externally supplied user notes retain provenance and are immutable in this workflow.

Generated results update non-destructively by analysis type. Modifying source records or user-authored content requires the confirmed-change flow.

## Visible UI Action Routing

FlowVerse-owned buttons, follow-up actions, related-issue links, and retained commands do not invoke hidden workflows. Each action captures its issue and artifact bindings before focus changes, opens or reveals Chat, displays a complete human-readable `@flowverse` prompt, and submits it automatically.

For a related issue already retained in the window, the visible prompt focuses that work item. For an uncached related issue, it asks to fetch that exact key. The clicked key is carried in the prompt; selecting a link never changes a hidden global issue.

Presentation-only interactions such as scrolling, expanding content, copying text, closing tabs, splitting editors, and native editor controls remain local.

## Architecture and Visualization

Architecture inspection is read-only and does not open a view. Ask explicitly to open, show, visualize, or inspect interactively when you want a layered diagram or another specialized visualization. Ordinary analysis never opens one automatically.

Selecting a diagram item changes only selection. It does not adopt a proposal, approve a write, or change the active Jira target.

## Plan and Apply Changes Safely

Ask for a preview before writing, for example:

```text
@flowverse Prepare exact Jira changes and a workspace file update for this issue. Show the complete plan and do not apply it.
```

Planning is non-destructive and requires no semantic confirmation. The preview contains the exact Jira operations and workspace file create/overwrite operations, current bindings, expiry, risks, a plan ID, and a single-use digest. Workspace targets must be inside a current workspace folder; traversal and out-of-workspace paths are rejected.

When the plan is correct, ask:

```text
@flowverse Apply the reviewed plan.
```

FlowVerse then presents the complete preview through native VS Code confirmation and applies only when you approve. Immediately before apply it rechecks the issue, context, architecture, Jira, and file-content revisions bound into the plan. Expired, used, rejected, mismatched, or stale plans cannot apply. Partial results identify each succeeded and failed operation, and mutations are never retried automatically.

This confirmation boundary applies to Jira creation and updates, links, transitions, workspace file creation or overwrite, user-authored content changes, and other external writes. Jira reads, explicit refresh, context reads, analysis, generated-analysis upserts, explicit visualization, and plan previews do not require semantic confirmation.

Simulator mutations exist only in memory and disappear when the simulator restarts or is reset.

## One-Release Compatibility

Legacy Jira aliases, the former generic analysis alias, and the four retained launch commands remain registered only for the first release containing this redesign. They are deprecated adapters, excluded from normal FlowVerse capability selection, and should not be used for new workflows.

During that release they preserve their existing public inputs and result fields while routing through the same context, refresh, visualization, and confirmation boundaries as the canonical capabilities. Retained commands display and automatically submit the corresponding Chat prompt; they do not directly run an analysis or open a view. Unsupported legacy requests are rejected rather than restoring hidden refresh, automatic view opening, or weaker write confirmation.

## Manual Copilot Smoke Test

Run this sequence first against the simulator, then repeat the relevant read-only steps against a permitted real Jira project with an installed production build:

1. Enter a plain unambiguous FlowVerse request and confirm VS Code routes it without intercepting an unrelated prompt.
2. Fetch a known issue and observe the five ordered progress stages.
3. Confirm the issue is retained without opening a report or optional diagram.
4. Request an analysis without repeating the key.
5. Confirm only the matching generated analysis changes and shows provenance, revision, and freshness.
6. Select cached and uncached related issues and confirm each visible automatically submitted prompt contains the clicked key.
7. Invoke each retained launch command and confirm it visibly submits a Chat prompt instead of running a hidden workflow.
8. Ask to inspect architecture, confirm no view opens, then explicitly ask to open a diagram and confirm it opens only then.
9. Plan a Jira mutation and a workspace file create or overwrite. Inspect and reject the preview once; create a fresh plan, approve it, and confirm revision and digest enforcement.
10. Restart the Extension Development Host and confirm prior full issue context is unusable until a visible fetch or refresh succeeds.

For the real-Copilot pass, use an issue you are authorized to read, avoid mutation unless you have a disposable test project, and verify that no prompt, output, or view exposes credentials or raw authentication details.

## Watching and Reloading During Development

The local Jira launch starts durable extension and webview watchers.

- After extension-host changes, wait for the TypeScript watcher to report no errors, then run **Developer: Reload Window** in the Extension Development Host.
- After webview-only changes, wait for Vite to finish, then run **Developer: Reload Webview**.
- After manifest, shared protocol, or cross-surface changes, run **Developer: Reload Window**.
- Use **Tasks: Terminate All Tasks** when you want to stop the simulator and watchers.

## Troubleshooting

### Chat cannot fetch Jira

1. Run **FlowVerse: Show Output**.
2. For local development, verify `curl http://127.0.0.1:4876/health` succeeds.
3. Verify the Extension Development Host was started from `flowverse-offline-debug.code-workspace`.
4. For production, run **FlowVerse: Set Jira Base URL** and **FlowVerse: Set Jira PAT**, then **Developer: Reload Window**.
5. Confirm the Jira base URL in the `flowverse.jiraBaseUrl` machine setting or the VS Code process environment.

### Retained context is stale

Stale context remains readable and never refreshes itself. Ask FlowVerse to refresh the issue or retrieve the latest Jira state. If refresh fails, continue using the preserved last-good context only when stale evidence is acceptable.

### A diagram did not open

Ordinary analysis intentionally does not open specialized views. Ask explicitly to open or visualize the desired diagram.

### Compatibility or readiness is `unknown`

This is expected when required before/after contracts, tests, Jira fields, or workspace evidence are missing. Attach or reference the missing evidence and rerun the capability. FlowVerse does not manufacture a confident answer from absent evidence.

### A plan will not apply

Create a fresh preview if the plan expired, was already used or rejected, or any bound Jira, context, architecture, or file revision changed. There is no automatic mutation retry.
