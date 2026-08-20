# FlowVerse Workspace Actions

FlowVerse exposes registered, typed operations to users, UI commands, Copilot, and other LLM clients. Every client uses the same action boundary; no client writes canvas components or source records directly.

Use `flowverse_list_actions` to retrieve the live catalog, `flowverse_describe_action` to inspect one strict input contract, and `flowverse_invoke_action` to execute it. Use exact stable IDs returned by FlowVerse.

## Execution modes

| Mode | Effect |
| --- | --- |
| `view` | Changes only filters, visibility, selection, or viewport state. |
| `analysis` | Reads the effective graph and returns deterministic results and evidence paths. |
| `preview` | Validates and simulates a complete change set without changing the active session. |
| `runtime-mutation` | Atomically updates the session overlay. Source files and external systems remain unchanged. |

Mutation results include validation errors, affected IDs, a change-set ID, and a human-readable summary. `requestId` makes retries idempotent. Unsupported nested properties, object types, relationship types, or endpoints are rejected instead of ignored.

## Shared catalog

| Action | Mode | Main parameters | Purpose |
| --- | --- | --- | --- |
| `workspace.types.list` | analysis | — | List declared entity concepts and relationship types. |
| `workspace.type.describe` | analysis | `typeId`, optional `domainId` | Describe one exact type and its observed use. |
| `workspace.entity.get` | analysis | `id` | Resolve one entity by canonical ID. |
| `workspace.relationship.get` | analysis | `id` | Resolve one relationship by ID. |
| `workspace.entity.neighbors` | analysis | `entityId`, optional `depth` | Return connected entities, relationships, and evidence paths. |
| `workspace.search` | analysis | `query`, optional `domainId`, `kind` | Search IDs, titles, text, kinds, badges, and fields. |
| `workspace.context.current` | analysis | — | Return the effective connected context and current selection. |
| `workspace.entity.create` | runtime-mutation | `entity`, optional `preview` | Create a session-only entity of a declared type. |
| `workspace.entity.update` | runtime-mutation | `entityId`, `patch`, optional `preview` | Update supported entity properties. |
| `workspace.entity.rename` | runtime-mutation | `entityId`, `title`, optional `preview` | Rename an entity without changing its stable ID. |
| `workspace.entity.reparent` | runtime-mutation | `entityId`, `parentId`, optional `preview` | Move an entity and its structural relationship atomically. |
| `workspace.entity.duplicate` | runtime-mutation | `entityId`, `newId`, optional `title`, `preview` | Create a temporary design alternative without copying relationships. |
| `workspace.entity.remove` / `restore` | runtime-mutation | `entityId`, optional `preview` | Remove or restore an entity through the session overlay. |
| `workspace.relationship.create` | runtime-mutation | `relationship`, optional `preview` | Create a validated relationship. |
| `workspace.relationship.update` | runtime-mutation | `relationshipId`, `patch`, optional `preview` | Update type, metadata, or endpoints. |
| `workspace.relationship.reconnect` | runtime-mutation | `relationshipId`, `sourceId` and/or `targetId` | Reconnect endpoints atomically. |
| `workspace.relationship.remove` / `restore` | runtime-mutation | `relationshipId`, optional `preview` | Remove or restore a relationship through the overlay. |
| `workspace.relationship.reverse` | runtime-mutation | `relationshipId`, optional `preview` | Reverse a relationship after validation. |
| `workspace.change.preview` / `apply` | preview / runtime-mutation | `mutations` | Validate and simulate or atomically apply a typed batch. |
| `workspace.history.list` | analysis | — | Return session change history and initiators. |
| `workspace.history.undo` / `redo` | runtime-mutation | optional `count` | Reverse or replay atomic change sets. |
| `workspace.session.diff` | analysis | — | Compare the effective session with its immutable baseline. |
| `workspace.session.reset` | runtime-mutation | — | Discard model changes and restore the source-loaded baseline. |
| `workspace.session.resetAll` | runtime-mutation | — | Discard model changes, scenarios, history, and view state. |
| `workspace.scenario.start` | runtime-mutation | optional `name` | Start an isolated temporary scenario. |
| `workspace.scenario.compare` | analysis | `scenarioId`, optional `otherScenarioId` | Compare alternatives without changing the active session. |
| `workspace.scenario.apply` / `discard` | runtime-mutation | `scenarioId` | Atomically apply or discard an alternative. |
| `workspace.view.filter` | view | `filter` | Apply a compound domain, type, field, text, relationship, or neighborhood filter. |
| `workspace.view.filter.clear` | view | optional `filterId` | Clear one filter or all filters. |
| `workspace.view.relationshipType.setVisible` | view | `relationshipTypeId`, `visible` | Hide or show a relationship type without deleting it. |
| `workspace.view.reset` | view | — | Clear temporary filters and visibility changes. |
| `workspace.analysis.dependencies` | analysis | `entityId`, optional `direction`, `depth` | Trace direct or transitive dependencies. |
| `workspace.analysis.paths` | analysis | `sourceId`, `targetId`, optional `maxDepth` | Find deterministic paths between exact IDs. |
| `workspace.analysis.impact` | analysis | optional `entityId`, `mutations`, `direction` | Simulate change impact without mutation. |
| `workspace.analysis.validate` | analysis | — | Return broken relationships and source validation findings. |
| `workspace.analysis.cycles` / `orphans` / `coupling` | analysis | action-specific optional limits | Detect cycles, disconnected roots, and highly connected elements. |
| `workspace.analysis.blockers` | analysis | optional `entityId`, `depth` | Return blocker relationships with ticket-ID evidence paths. |
| `workspace.analysis.crossDomainImpact` | analysis | `entityId`, optional `direction`, `depth` | Traverse only explicit or session-confirmed cross-domain mappings. |

The catalog also exposes the established `diagram.*` selection, focus, fit, zoom, layout, edge, minimap, and refresh actions. Architecture and Agile Delivery aliases such as `architecture.dataflow.reconnect`, `architecture.impact.analyze`, `agile.ticket.create`, and `agile.blockers.analyze` delegate to the same shared operations.

## AI-assisted work breakdown

`agile.ticket.breakdown` accepts an exact `parentId`, an optional maximum, optional user constraints, and optional strict proposals for deterministic clients. Without supplied proposals, the extension asks an available Copilot model for strict JSON. The result is validated before one atomic change set creates draft work items and parent-child relationships.

Generated items use collision-safe provisional IDs and session-only provenance. They do not invent assignees, sprints, dates, or estimates, and they may reference architecture elements only through explicit or session-confirmed mappings. One undo removes the entire generated breakdown.

Example prompts:

- `@flowverse preview a REST relationship from architecture:service-a to architecture:service-b`
- `@flowverse show everything downstream of architecture:customer-api`
- `@flowverse compare this scenario with the loaded baseline`
- `@flowverse create at most five sub-tasks of agile-delivery:PACIFIC-1234`
- `@flowverse hide blocks relationships, then clear that filter`
- `@flowverse show confirmed cross-domain impact from agile-delivery:PACIFIC-1234`

## Runtime flow

```text
source connectors -> immutable baseline -> session overlay -> effective composition
                                                        -> canvas, details, search, analysis

LLM / UI -> discover typed action -> validate or preview -> atomic overlay change
                                                -> change event -> synchronized views
```

Reload and reset recover the exact source-backed baseline. Future persistence belongs behind an explicit authoritative-source adapter and is not part of these workspace actions.
