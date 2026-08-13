# FlowVerse for VS Code

[简体中文](README-zh_CN.md)

## Introduction

FlowVerse is an AI-native engineering platform for connecting the perspectives needed to launch, operate, and continuously evolve complex products at speed.

Configured and authorized domain knowledge forms a connected, source-backed context shared by engineers, tools, LLMs, and agents. An LLM-command design pattern exposes bounded semantic operations over that context, allowing models and agents to integrate naturally through the same platform capabilities used by the product rather than through a separate AI layer. Engineering teams extend the shared foundation with domain modules that define how their own data, relationships, validation, and presentation integrate. Once configured, the platform provides one navigable product view across lifecycle domains, ranging from product strategy, requirements, delivery work, source code, and models to PLM records, digital twins, asset registries, industrial automation systems, deployed software, integrations, and operational data flows.

Using this connected knowledge across authorized domains, AI agents working through FlowVerse can answer questions and generate grounded solution proposals. FlowVerse can then present and connect those proposals and their approved outcomes—through governed domain commands—to the relevant records across domains, giving teams a traceable path from context and analysis to decisions and results. When those domains are registered, the same pattern connects proposals to requirement changes, PLM and model revisions, digital-twin scenarios, industrial automation configurations, asset context, and operational evidence—not only to software changes.

## Smart Factory Demo

### Background

The included demo models a manufacturer whose in-house Kubernetes platform operates CNC machines, robots, and assembly lines. Industrial equipment publishes OPC UA and MQTT telemetry into a Kafka event backbone; platform services turn those signals into machine state, anomaly alerts, failure predictions, maintenance work orders, operator notifications, and production-impact records. External CMMS, ERP/MES, supplier, equipment-manufacturer, and notification systems complete the operational landscape.

The same product is represented from two independently owned perspectives. The System Architecture domain describes services, modules, interfaces, infrastructure, and operational data flows. The Agile Delivery domain describes a five-level portfolio of epics, stories, tasks, bugs, and a spike. Explicit cross-domain links connect delivery intent to the systems and modules it scopes or implements without copying either source record. All organizations, teams, issue keys, endpoints, and operational content are fictional.

### Use the released demo

1. Install the latest `releases/flowverse-vscode-vX.Y.Z.vsix` from the release repository with **Extensions: Install from VSIX...**.
2. Open `resources/examples/` as a folder in VS Code. FlowVerse discovers its `unified-worldview.instance.json` and the referenced architecture, delivery, and cross-domain records.
3. Open the FlowVerse activity-bar view, or use the Command Palette to run one of these commands:

   - **FlowVerse: Open System Architecture Canvas** for the operational technology and software landscape;
   - **FlowVerse: Open Agile Delivery Canvas** for delivery hierarchy, dependencies, blockers, evidence, and readiness;
   - **FlowVerse: Open Unified Worldview Canvas** for the connected cross-domain view.

4. Select cards, relationships, or data-flow steps to inspect their source-backed details. Use `@flowverse` to navigate the same context, answer questions, or focus a named entity when Copilot Chat is available.

For extension development, open this source repository, select **Run FlowVerse with Smart Factory Example** in Run and Debug, and press `F5`. The Extension Development Host opens the same dataset directly.

Read the [Smart Factory Operations Example](docs/smart-factory-operations-example.md) for the complete system story, service responsibilities, operational flows, delivery model, and cross-domain traceability.

## Source-backed Architecture

FlowVerse can turn source material into structured architecture knowledge and immediately render the result. The canvas is not an isolated AI-generated picture: the details workbench retains origin, acquisition method, confidence, review state, evidence references, and ownership alongside the selected system. The `@flowverse` agent can open the relevant view while the explorer, canvas, details, and chat remain synchronized.

![AI-assisted architecture authoring with source lineage, architecture canvas, explorer, and FlowVerse agent](docs/screenshots/architecture-authoring-by-AI.png)

*AI-assisted architecture acquisition remains traceable. The selected Telemetry Ingestion Service exposes its evidence and lineage while the generated architecture stays navigable on the canvas.*

### Connected Architecture Landscape

The System Architecture canvas provides a portfolio-scale view of systems, internal modules, external platforms, APIs, event channels, databases, and their relationships. Renderer-owned card and edge conventions keep the model readable, while the explorer and dynamic legend provide stable navigation and visual meaning. The lower details workbench summarizes scope and connectivity from the same loaded records.

![Smart Factory system architecture landscape with services, modules, integrations, explorer, legend, and details](docs/screenshots/architecture-diagram.png)

*The complete smart-factory landscape connects industrial equipment, Kubernetes services, event streaming, operational data, and external vendors in one architecture view.*

## AI-guided Navigation

The Copilot-native `@flowverse` participant controls the same canvas commands used by the explorer and UI. A user can ask for a named module, service, relationship, or data flow; FlowVerse resolves the source-backed entity, highlights it, focuses the viewport at a bounded zoom, and opens its details. The agent does not create a parallel diagram or a second copy of the architecture.

![FlowVerse Copilot request highlighting and focusing the Notification Policy module](docs/screenshots/highlight-module-by-copilot.png)

*Natural-language navigation lands on the Notification Policy module inside its owning service and exposes the module's source-backed architecture details.*

## Operational Data Flows

Architecture becomes useful when teams can inspect how work actually moves through it. The Data Flows tab aggregates every declared path in the current scope, including flow type, owner, participating entities, relationship coverage, and ordered step count. The register is sortable and its columns are resizable, so the same view works for portfolio review and focused investigation.

![Architecture data-flow register beside the smart-factory architecture canvas](docs/screenshots/architecture-list-all-dataflows.png)

*The architecture-wide data-flow register turns a dense topology into an operational index of telemetry, maintenance, API, and event-driven paths.*

Selecting a flow highlights its full route. Selecting an individual step narrows the canvas to the exact source, target, and relationship for that part of the execution path, dims unrelated elements, and keeps the ordered path visible in the details table.

![A selected data-flow step highlighted between the notification client and provider](docs/screenshots/highlight-dataflow-step.png)

*Step-level focus connects an ordered business or technical flow to the precise architecture interaction that implements it.*

## Delivery in Engineering Context

The Agile Delivery canvas presents epics, stories, tasks, bugs, and spikes as a delivery hierarchy instead of a flat issue list. Solid top-to-bottom relationships express parent and child structure; color-coded dashed relationships preserve dependencies, blockers, validation, implementation, and other delivery semantics without distorting hierarchy levels. The explorer and legend switch to the active delivery domain automatically.

![Agile Delivery hierarchy with epics, stories, tasks, bugs, relationships, explorer, legend, and details](docs/screenshots/agile-delivery-diagram.png)

*The delivery canvas makes scope decomposition and work-item relationships visible across multiple hierarchy levels while retaining the original Jira-style records.*

### Structured Work-item Reports

Selecting an item opens a structured report rather than a generic property dump. Overview, Relationships, Evidence, Comments, AI Rating, and Source separate delivery identity, ownership, planning context, requirements, acceptance and completion evidence, discussion, assessment, and lineage. This gives product, engineering, architecture, quality, and operations teams a shared review surface without changing the source issue.

![Structured Agile work-item report for the Factory platform epic](docs/screenshots/agile-task-report.png)

*The report keeps operational delivery context beside the hierarchy, so reviewers can understand both the selected item and where it sits in the broader plan.*

### Visible Dependencies and Blockers

Logical delivery relationships are rendered independently from the parent-child hierarchy. Selecting a `blocks` edge highlights the blocker and blocked item, dims unrelated work, and explains the relationship's direction and delivery effect in the details panel. Teams can see a gating defect in context instead of reconstructing it from linked-issue fields.

![Selected blocking relationship between a bug and delivery task](docs/screenshots/agile-task-visualize-block-relationship.png)

*The selected bug blocks the CMMS integration task; the canvas and details view show both endpoints, direction, and gating effect.*

## Connected Product Context

Product knowledge, decisions, and outcomes are distributed across independently owned domains, each with its own records, tools, and authority. FlowVerse connects these perspectives into one source-backed worldview while preserving that domain ownership. The bundled development example demonstrates this extensible model by connecting a smart-factory System Architecture domain with an Agile Delivery domain.

![Cross-domain implements relationship from Agile task FACTORY-145 to the Notification Provider Client architecture module](docs/screenshots/link-agile-task-to-architecture-module.png)

*The bundled example links delivery intent to architecture implementation without duplicating either record. The same model can connect any addressable elements supplied by registered domains.*

Teams can move from an epic or task to the service or module it implements, follow the affected data flow, inspect evidence, and trace the connected decisions and outcomes. Engineering and delivery leaders can review the complete chain of impact, dependencies, forecasts, and results across domains. Teams can extend the same pattern with simulation domains, connecting assumptions and inputs to simulation results, evaluated product elements, and the decisions those results inform.

FlowVerse combines a shared interactive canvas, domain-specific visualizations, validated source records, Jira-aware delivery intelligence, and a Copilot-native `@flowverse` agent. The same interaction model-explore, select, focus, inspect, and trace-applies across domain canvases and the composed worldview.

## Explainable Delivery Readiness

When a work item has no assessment, the AI Rating tab explains which source-backed factors will be evaluated and offers one restrained **Rate work item** action. The assessment uses the active delivery profile and current work-item evidence; it does not silently modify the Jira-style source record.

![AI Rating tab before generating a work-item readiness assessment](docs/screenshots/agile-task-using-AI-to-rate-task.png)

*The rating action is available in context beside the selected task, its blocker, ownership, priority, and lineage.*

The result connects an overall readiness rating to its component factors, thresholds, weights, and rationale. Color-coded bands distinguish ready, mostly ready, needs refinement, and at-risk results. Priority recommendations translate the weakest factors into concrete next actions, while the generated assessment remains session-scoped unless it was explicitly supplied by source data.

![AI work-item readiness result with composite score, five factor scores, rationale, and priority recommendations](docs/screenshots/agile-task-using-AI-to-rate-task-result.png)

*The example scores `FACTORY-122` at 83/100: purpose, scope, acceptance, ownership, and dependency/risk readiness remain individually explainable instead of being hidden behind one AI score.*

## Documentation

The README is the product introduction and visual walkthrough. Use the focused guides below for the platform mechanics, extension points, AI workflows, and contributor operations.

| Guide | Perspective | Best for |
| --- | --- | --- |
| [Platform Model and Canvas](docs/platform-and-canvas.md) | Product model, canvas types, shared interactions, details, and source precedence | Product teams, architects, and delivery teams |
| [Domain Integration](docs/domain-integration.md) | Domain packages, connectors, validation, resources, and data resolution | Domain and platform engineers |
| [AI-native Workflows](docs/ai-native-workflows.md) | LLM-command integration, agent navigation, analysis, and governed actions | Agent builders and FlowVerse users |
| [Development and Releases](docs/development-and-releases.md) | Setup, validation, packaging, CI, releases, and repository layout | Contributors and maintainers |

### Supporting References

- [Smart Factory Operations Example](docs/smart-factory-operations-example.md) - end-to-end example connecting delivery work with architecture and operational flows.
- [AI Agent User Guide](docs/AI_AGENT_USER_GUIDE.md) - setup, commands, approvals, capability verification, and troubleshooting.
- [Diagram Action Framework](docs/architecture/diagram-action-framework.md) - shared command contract for deterministic canvas navigation and actions.
- [Architecture Design](docs/architecture-design.html) - interactive technical architecture reference.
