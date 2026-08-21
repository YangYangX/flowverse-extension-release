# Platform Model and Canvas

This guide explains how FlowVerse composes source-backed product knowledge and presents it through a shared canvas and details workbench.

## Product Model

FlowVerse follows one composition chain:

```text
Shared data contracts
  -> Domain worldview.instance.json
    -> Domain records (*.architecture.json, *.jira.json, ...)
      -> Canvas projection
```

`unified-worldview.instance.json` composes those independently owned domains without copying their records. It declares:

- the shared data contracts;
- each domain worldview;
- cross-domain relationship types;
- link sets that reference `*.links.json` assertions.

Every canvas is a projection of the same loaded source data. Opening a domain canvas, the unified canvas, the explorer, or the details panel does not create another authoritative copy.

The JSON contracts remain semantic and renderer-neutral. They define concepts, records, types, relationships, views, and layout intent. Card design, colors, icons, edge rendering, selection behavior, toolbars, and minimap styling belong to this extension as the default rendering application.

## Current Canvases

### Unified Worldview

The Unified Worldview canvas composes all enabled domains and their cross-domain links on one generic canvas. A delivery item can therefore be connected to the service, module, interaction, or other addressable architecture element that it scopes, implements, or requires.

### System Architecture

The System Architecture canvas renders systems, reference systems, modules, interactions, infrastructure, and ordered data flows. Visual mapping is driven by the item type stored in the architecture data, so the same type has the same renderer-owned color and card treatment throughout the canvas.

### Agile Delivery

The Agile Delivery canvas renders Jira-style work items from `*.jira.json` records. Its default organization-chart layout comes from the domain worldview's semantic view configuration: parents occupy one hierarchy level and siblings share the next level. Non-structural relationships such as blocks, depends on, implements, relates to, and has defect remain visible as semantic edges.

Its details view separates operational context from relationships, delivery evidence, discussion history, AI rating, and source lineage. The Agile Delivery worldview owns the available assessment profiles—including factor names, descriptions, weights, aggregation, and rating bands—while the extension executes and renders the selected profile. Generated ratings remain session-scoped unless an assessment is explicitly present in source data.

## Shared Canvas Behavior

All domain and unified views use the same canvas engine in `src/flowverse/workbench/` and the same canvas surface in `src/webview/features/domainCanvas/`.

- Single-click highlights a card or edge; double-click highlights and focuses it.
- Selecting an edge highlights both endpoints and dims unrelated cards and edges.
- Explorer selection opens the required canvas when necessary, then applies the same highlight-and-focus policy.
- Selecting a data flow highlights the full path; selecting a step narrows the highlight to that step.
- Bundle mode remains compatible with edge and data-flow selection.
- The optional edge data-flow popover is off by default.
- Zoom `100%` means React Flow actual size. Fit, zoom presets, actual size, minimap, routing, bundling, and reset use shared controls.
- The minimap is hidden by default and uses the active domain's renderer theme.
- The FlowVerse Legend changes with the active canvas and uses the same node and edge mappings as that canvas.

The FlowVerse Explorer uses deferred tree loading. Expanding a domain only loads its tree children; clicking the domain opens its canvas and details. Closing an editor does not clear explorer data.

## Details Workbench

The shared details workbench provides **Overview**, **Data Flows**, **Source**, and **Review** tabs. Domain modules supply domain-specific sections without replacing the workbench shell.

- Overview values come from the selected source record or derived counts over the loaded model.
- Data-flow selections show ordered steps; systems, modules, and references show a sortable register of all involved flows.
- Source presents provenance and lineage.
- Review presents evidence-backed governance, risk, and verification information.
- Enterprise tables support sorting and drag-resizing columns without changing total table width.

For Agile Delivery, the same shell provides **Overview**, **Relationships**, **Evidence**, **Comments**, **AI Rating**, and **Source**. Overview contains operational identity and planning context; Evidence owns requirements, acceptance and completion criteria, risks, and open decisions; Comments preserves source-backed authorship and timestamps. AI Rating shows the composite score, color-coded factor bands, rationale, and prioritized recommendations without modifying the source work item.

Except for necessary generic presentation mappings such as labels, tones, and formatting, the details application does not manufacture business information.

## Source and Assertion Precedence

Workspace or explicitly loaded project data takes precedence over bundled defaults for the same source. Bundled resources are a safe fallback, not a competing runtime dataset.

Equivalent cross-domain relationships use this authority order:

1. session-confirmed user assertions;
2. explicit workspace link-set assertions;
3. runtime-inferred relationships.

Relationships with a different type, direction, or endpoint pair remain separate assertions.
