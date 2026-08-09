# FlowVerse Launch Intelligence — Native VS Code Implementation Plan

## Summary

Implement the ten-feature portfolio as a native VS Code and Copilot experience:

- **Left Primary Sidebar:** active launch scope, Jira context, architecture layers, decisions.
- **Editor area:** Jira reports, solution comparison, and architecture diagrams.
- **Right side:** the native Copilot Chat view with `@flowverse`.
- **Bottom Panel:** native analysis evidence, findings, gates, run status, and the existing FlowVerse Output channel.

Stable VS Code APIs cannot force an extension view into the Secondary Sidebar, so FlowVerse will use Copilot Chat there and avoid experimental APIs. Users remain free to rearrange contributed views. [VS Code sidebar guidance](https://code.visualstudio.com/api/ux-guidelines/sidebars)

Webviews become display-and-selection surfaces. They contain no application navigation links, approval controls, forms, or write actions.

## Native Workbench Architecture

### Shared session controller

Add one extension-host `FlowVerseSessionController` as the authoritative state for:

- Active Jira issue and delivery scope.
- Active unified worldview.
- Selected system, module, interaction, flow, risk, or Jira item.
- Current analysis artifacts and source revisions.
- Adopted solution and pending change plan.
- Active Jira report and architecture editor sessions.

Commands, Tree Views, LM tools, Copilot Chat, and webviews consume this controller instead of maintaining parallel state.

Use context keys such as:

- `flowverse.hasActiveIssue`
- `flowverse.hasArchitecture`
- `flowverse.hasSolutions`
- `flowverse.hasImpact`
- `flowverse.hasPendingPlan`
- `flowverse.hasSelection`

These control editor toolbar actions, Tree View actions, and command availability.

### Left FlowVerse sidebar

Keep the existing `flowverse` Activity Bar container and `flowverse.layers`. Add:

1. `flowverse.launchExplorer`

   - Active Jira issue and epic.
   - Requirements and open questions.
   - Solution options and adopted decision.
   - Impacted architecture grouped by system.
   - Delivery plan and blocker summary.
   - Verification and readiness status.

2. `flowverse.layers`

   - Base architecture.
   - Jira cards and badges.
   - Proposal and impact overlays.
   - Confidence, relationship, impact-kind, and solution filters.

Tree selection changes the active scope and focuses the editor. Actual actions live in view-title or item-context commands rather than fake button-like tree items, following native Tree View guidance. [VS Code Views guidance](https://code.visualstudio.com/api/ux-guidelines/views)

### Editor webviews

Retain webviews only where VS Code has no native equivalent:

- Jira report.
- Solution comparison.
- Embedded mini-topologies.
- Full architecture/impact canvas.
- Coverage and release heatmaps.

Rules:

- Pointer interaction may select a diagram node, card, path, or report section.
- Selection only posts a typed `selectionChanged` message to the host.
- Selection never directly opens another view, writes data, runs AI, or approves anything.
- Routing happens through registered VS Code commands.
- Report actions move to the editor title toolbar and Command Palette.
- Replace the HTML report TOC with the native Launch Explorer.
- Jira Markdown links post `openExternalRequested`; the extension validates the URL and calls `vscode.env.openExternal`.
- Remove HTML forms, navigation anchors, application CTA links, and approval buttons.
- Preserve ready/loading/error/diagnostic messages and restored-panel compatibility.

### Right-side Copilot Chat

Register one native participant:

- Name: `@flowverse`
- Full name: `FlowVerse Launch Intelligence`
- Sticky in the current Chat session.
- Uses the model selected by the user through `request.model`.
- Uses native streaming progress, references, follow-ups, and command buttons.
- Invokes registered LM tools with `request.toolInvocationToken`, so tool activity and confirmations appear in Copilot Chat.

Slash commands map directly to the ten features:

- `/brief`
- `/solutions`
- `/impact`
- `/plan`
- `/blockers`
- `/compatibility`
- `/tests`
- `/drift`
- `/readiness`
- `/why`

The participant follows the existing prompt-loader pattern under `prompts/`. Implement the tool loop locally without adding `@vscode/chat-extension-utils` or another production dependency. VS Code officially supports participant commands, follow-ups, command buttons, selected models, and tool invocation. [Chat Participant API](https://code.visualstudio.com/api/extension-guides/ai/chat)

Generic Copilot Agent Mode can continue invoking the LM tools without explicitly selecting `@flowverse`.

### Bottom panel

Use the native Tree View, `flowverse.analysis`, for:

- Current analysis run and freshness.
- Evidence grouped by Jira, architecture, repository, test, user, and model.
- Findings grouped by severity.
- Passed, failed, and unknown readiness gates.
- Pending and completed change operations.
- Partial-data and validation issues.

Selecting evidence reveals its native JSON document, field, Jira report, or architecture selection. View toolbar commands provide refresh, reveal source, copy reference, and open Output.

Reuse the existing FlowVerse Output channel for technical logs, timings, cache events, tool calls, validation summaries, and failure diagnostics. Never log prompts containing Jira descriptions, credentials, or sensitive content.

## Analysis and Agent Contracts

### Knowledge pack

Bundle the architecture JSON, Core Ontology, domain ontologies, worldview instances, unified worldview, and link sets inside the extension.

Resolution order:

1. Explicit `flowverse.worldviewPath`.
2. Valid workspace knowledge pack.
3. Bundled read-only plugin knowledge pack.

Build a semantic `ArchitectureKnowledgeIndex` containing systems, modules, endpoints, interactions, incoming/outgoing edges, data-flow membership, risks, review questions, ownership, and criticality. Do not reason over React Flow nodes.

### Versioned artifacts

Introduce additive artifacts without modifying canonical architecture schemas:

- `LaunchBriefArtifact`
- `ScenarioSetArtifact`
- `ImpactArtifact`
- `DeliveryPlanArtifact`
- `DependencyRadarArtifact`
- `CompatibilityArtifact`
- `CoverageDesignArtifact`
- `DriftArtifact`
- `ReleaseAssessmentArtifact`
- `DecisionRecord`

Every artifact contains:

- `schemaVersion`, `kind`, `id`, and `featureId`
- Exact Jira and architecture scope references
- Source revisions and timestamps
- Evidence references
- Claims with `observed`, `derived`, `proposed`, `approved`, `rejected`, or `superseded`
- Deterministic confidence
- Findings and partial-data issues
- Generator version and freshness state

Drafts stay in extension workspace storage. Tracked decision or analysis files are exported only through an approved workspace change plan.

### LM tools

Keep the existing seven Jira tools and schemas unchanged. Add:

1. `flowverse_analyse`

   - Read-only.
   - Uses a versioned feature discriminator for the ten analyzers.
   - Returns a validated artifact and native editor projection.

2. `flowverse_plan_changes`

   - Converts an adopted proposal into exact Jira or workspace operations.
   - Produces a visible before/after plan, digest, source revisions, risks, and one-use token.
   - Makes no changes.

3. `flowverse_apply_changes`

   - Requires the valid token.
   - Rechecks Jira versions and file hashes.
   - Uses native `prepareInvocation` confirmation.
   - Executes Jira and workspace operations as a visible saga with partial-failure reporting.

The native confirmation UI remains mandatory for tool writes. [VS Code Language Model Tool API](https://code.visualstudio.com/api/extension-guides/ai/tools)

## Ten-Feature Delivery

### P0 — Launch cockpit

1. **Launch Brief Compiler:** raw specification or Jira issue to classified requirements, gaps, architecture scope, and Jira proposal.
2. **Solution Scenario Studio:** two or three evidence-backed solution options with mini-topologies, risk, blockers, assumptions, and delivery ranges.
3. **Change Impact Atlas:** bounded direct and propagated impact through systems, modules, APIs, messages, files, infrastructure, and flows.
4. **Architecture-Aware Delivery Planner:** adopted solution to system/module-oriented Jira work packages and verification obligations.
5. **Dependency and Critical-Path Radar:** confirmed and predicted blockers, cycles, downstream effect, owners, and topology- versus estimate-based paths.

### P1 — Break-nothing intelligence

6. **Contract and Flow Compatibility Guardian:** before/after semantic compatibility for APIs, messages, files, protocols, and data flows.
7. **Acceptance-to-Test Coverage Designer:** requirement × architecture path × test-obligation coverage.
8. **Intent-to-Implementation Drift Sentinel:** approved intent versus architecture revisions, workspace changes, and available test evidence.

### P2 — Lifecycle governance

9. **Release Confidence Command Center:** explainable `Ready`, `Needs review`, or `Not ready` gates without an opaque AI score.
10. **Living Product Decision Memory:** approved and superseded decisions connected to Jira, architecture, risks, evidence, and release outcomes.

## Implementation Sequence

1. Preserve a baseline by running current plugin tests, typechecks, builds, and architecture validators.
2. Add the bundled knowledge resolver, strict validation, semantic architecture index, source revisions, and the missing `backward` propagation support.
3. Add shared session, analysis registry, artifact storage, command router, and controller events.
4. Add native Launch Explorer and Analysis Tree Views; migrate layered configuration to native checkboxes and commands.
5. Register `@flowverse`, its slash commands, prompt templates, tool loop, follow-ups, streamed references, and command buttons.
6. Convert report and architecture webviews to display-and-selection-only protocols.
7. Deliver the P0 analyzers and editor projections.
8. Generalize the existing Jira approval flow into digest- and revision-bound change planning.
9. Deliver P1 safety analyzers, including the read-only local repository/test evidence adapter.
10. Deliver release assessment and decision memory.
11. Remove obsolete webview action messages only after restored-panel compatibility tests pass.
12. Create the required workspace implementation, quality, delivery, and service-manual evidence during execution.

## Native Commands

Provide commands usable from the Command Palette, editor toolbar, Tree View menus, Chat buttons, and keybindings:

- `FlowVerse: Set Active Jira Issue`
- `FlowVerse: Ask Copilot`
- `FlowVerse: Compile Launch Brief`
- `FlowVerse: Compare Solutions`
- `FlowVerse: Open Impact Atlas`
- `FlowVerse: Focus Selected Architecture`
- `FlowVerse: Create Delivery Plan`
- `FlowVerse: Review Pending Changes`
- `FlowVerse: Reveal Evidence`
- `FlowVerse: Assess Release Readiness`
- `FlowVerse: Explain This Decision`
- `FlowVerse: Show Output`

Inputs use `showQuickPick` for bounded choices and `showInputBox` only for Jira keys, JQL, or raw specification text. Writes invoked outside Chat must still route through the same planned-change confirmation boundary.

## Test Plan

- Manifest tests for chat participant, slash commands, LM tools, commands, view IDs, menus, and context keys.
- Session tests for multiple reports/canvases and deterministic active-selection policy.
- Chat tests for intent routing, selected-model usage, history, tool tokens, cancellation, streaming, follow-ups, command buttons, and errors.
- Adapter parity tests proving Chat, native commands, LM tools, and Tree Views produce the same service result.
- Protocol tests proving webviews cannot initiate AI execution or mutations.
- Static checks rejecting application `href` routing, HTML input controls, and webview approval actions.
- Native URL-routing tests for safe HTTP/HTTPS links and rejected schemes.
- Analysis tests for exact IDs, evidence, confidence, stale revisions, partial sources, cycles, caps, and all propagation directions.
- Approval tests for rejection, expiration, token reuse, changed Jira versions, changed file hashes, partial failure, and recovery.
- Extension-host smoke tests for activation, `@flowverse`, Tree Views, Chat tool invocations, report/diagram focus, Output, and restored editors.
- Regression gates: current plugin tests, extension and webview typechecks, production build, strict architecture validation, unified-worldview validation, and schema parity.

## Assumptions

- Use the stable native layout: left FlowVerse sidebar, editor-area visuals, Copilot Chat on the right when the user’s workbench is arranged that way, and bottom analysis/output.
- Do not use VS Code’s proposed Secondary Sidebar contribution API.
- The first production milestone stops at an approved Jira plan; source edits and PR creation remain later opt-in work.
- Existing Jira commands, LM tools, raw JSON report, service preview, and layered architecture remain compatible.
- Diagram selection is allowed inside webviews; all subsequent actions are native commands or Copilot interactions.
- No new production dependency is introduced without separate approval.
