# FlowVerse Diagram Action Framework

The architecture canvas is controlled through one shared action path:

1. Register a capability in `src/extension/features/diagramActions/registry.ts`.
2. Resolve user-facing labels or stable ids through `DiagramReferenceResolver`.
3. Execute through `DiagramActionExecutor`.
4. Surface the same action through LM tools, VS Code commands, explorer gestures, or chat links.

This keeps direct canvas interactions and AI-assisted interactions aligned. A chat result, a command palette action, and a tool call should all produce the same state update and the same validation behavior.

## Core Pieces

- `DiagramActionRegistry` stores stable action ids, command ids, target types, categories, safety metadata, aliases, and examples.
- `DiagramActionContextProvider` reads the active layered canvas and can open the default unified worldview when an action is allowed to do so.
- `DiagramReferenceResolver` maps natural-language targets to graph references. Ambiguous results return clickable command arguments instead of guessing.
- `DiagramActionExecutor` validates arguments, checks availability, resolves targets, and calls the `LayeredController` APIs.
- `registerDiagramActionCommands` exposes `flowverse.diagram.execute` plus one command per action, using versioned arguments.

## Public Surfaces

Current LM tools stay stable:

- `flowverse_open_diagram` opens a resolved domain or unified worldview canvas.
- `flowverse_control_diagram` accepts the existing kebab-case canvas actions and forwards them to the shared executor.

Clickable chat actions should use:

```json
{
  "version": 1,
  "actionId": "diagram.focusNode",
  "args": {
    "nodeId": "vendor-adapter-service"
  }
}
```

The command id is `flowverse.diagram.execute`.

## Adding An Action

Add the registry definition first. Include:

- stable `id` and `commandId`
- optional legacy `toolAction`
- `category`
- supported `targetTypes`
- `channels`
- safety and availability metadata
- required input fields
- examples and aliases

Then implement the action in `DiagramActionExecutor`. Prefer calling `LayeredController` methods instead of posting webview messages directly.

Actions that only change view state do not require semantic or native confirmation. Actions that would mutate Jira, files, or retained context should not be added to this framework without a separate planning and confirmation path.
