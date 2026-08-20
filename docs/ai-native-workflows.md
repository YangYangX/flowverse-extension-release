# AI-native Workflows

FlowVerse gives LLMs and agents access to the same source-backed context and bounded semantic commands used by the explorer, canvas, and details workbench. Agent workflows therefore navigate and act on the product model without creating a parallel AI-specific copy of its data.

## LLM-command Pattern

The command boundary keeps model reasoning separate from platform execution:

- FlowVerse resolves entities and relationships from the active source-backed session.
- Human-readable requests map to bounded navigation, analysis, and mutation commands.
- The extension executes those commands through the same application services used by the UI.
- Mutations remain subject to preview and confirmation rather than being applied directly by model output.

## Copilot-native Launch Intelligence

`@flowverse` uses the model selected in VS Code. It does not contribute another model provider or bypass enterprise model policy.

Start with:

```text
@flowverse fetch FACTORY-122
```

FlowVerse can acquire Jira evidence, open a structured issue report, inspect architecture impact, compare solutions, plan delivery, analyze blockers, assess compatibility and readiness, generate tests or diagrams, and navigate the relevant canvas. UI actions that invoke agent work are translated into visible human-readable prompts and submitted through the same participant workflow.

Read-only analysis and navigation do not require semantic confirmation. Jira mutations and workspace file creation or overwrite use a preview, a revision-bound single-use plan, and native confirmation immediately before apply.

Full issue context and generated report sections live in the current VS Code window. Reopening a retained report does not silently fetch Jira data; request a refresh when current server state matters.

Fetching a work item acquires its bounded descendant hierarchy (three levels and 75 descendants by default), merges it with source records already available from the workspace, opens the Agile Delivery canvas, and focuses the requested item. Additional fetches compose independent roots into the same in-memory session. Clearing a fetched root—or all fetched work—removes only session-acquired records; workspace files remain authoritative and unchanged.

See the [AI Agent User Guide](AI_AGENT_USER_GUIDE.md) for setup, approvals, capability verification, and troubleshooting. The [Diagram Action Framework](architecture/diagram-action-framework.md) describes the shared command boundary used for canvas actions.
