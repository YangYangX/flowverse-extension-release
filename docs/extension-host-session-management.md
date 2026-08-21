# Extension Host Session and Knowledge Architecture

## Purpose

This document describes the implemented FlowVerse Extension Host session model. It separates UI sessions from knowledge ownership and records how workspace files, authoritative providers, runtime contributions, and bundled demo data are resolved through one source-independent framework.

Local JSON is supported as an adapter. It is not the platform's state architecture or assumed source of truth.

## Session model at a glance

FlowVerse uses complementary sessions with different responsibilities:

```text
Workspace override ─┐
Authoritative source ├─> provider registry ─> persistent/runtime knowledge store
Bundled fallback  ──┘                              │
                                                   v
                                      normalized composition/read models
                                                   │
                         ┌─────────────────────────┴────────────────────────┐
                         v                                                  v
                  canvas session                                  Launch/AI session
             projection and UI state                         analysis and interaction state
```

| Session or model | Owns | Lifetime |
| --- | --- | --- |
| Persistent Knowledge Session | Stable, reusable knowledge and provider metadata | Rehydrated from abstract persistence across Extension Host lifetimes |
| Runtime Knowledge Session | Dynamic authoritative, temporary, and derived knowledge | Extension Host lifetime |
| `BridgeComposition` | Disposable normalized read model assembled from session knowledge | Cached and regenerated when inputs change |
| Canvas session | One live canvas panel, perspective, selection, command queue, and webview readiness | Until its panel closes |
| Launch/AI session | Active context and analysis interaction state | Controller/window lifetime |

The sessions do not copy source records into view-specific stores. Canvas, Explorer, Details, diagram tools, and Launch Intelligence resolve knowledge through the same provider-backed platform and derive their own disposable projections.

## Source resolution

[`KnowledgeProviderRegistry`](../src/extension/platform/knowledge/providerRegistry.ts) applies one deterministic policy for every resource:

1. A matching workspace resource is an explicit override.
2. Otherwise, registered authoritative providers are queried.
3. If allowed and no higher tier resolves the resource, a fallback provider may supply local or bundled demo data.

Providers within a tier are ordered by descending priority and then registration order. A failure is retained as a provider attempt and does not prevent the next provider from resolving the resource. Callers do not branch on workspace, MCP, agent, memory, network, or file source kinds.

The default platform registers:

- `workspace-json` as a workspace-override adapter.
- `bundled-json` as a fallback adapter over the read-only knowledge pack.

An authoritative provider is registered through the same stable interface. Registration invalidates affected session cache so the next request applies the new precedence order.

Every resolved resource retains:

- Resource type and canonical ID.
- Provider ID and provider kind.
- Persistent or runtime ownership.
- Workspace-override, authoritative, fallback, temporary, or derived disposition.
- Fresh/cached state and load timestamp.
- Optional version, revision, expiration, and refresh timestamp.
- Provenance references.

## Persistent Knowledge Session

[`PersistentKnowledgeSession`](../src/extension/platform/knowledge/persistentKnowledgeSession.ts) owns relatively stable, reusable knowledge such as shared definitions, schemas, worldviews, settings, and provider configuration.

Its Redux Toolkit store uses a normalized resource map keyed by `resourceType:canonicalId`, plus request status and provider-attempt state. The session supports:

- Asynchronous lazy resolution.
- In-memory caching and explicit refresh.
- Per-resource and whole-session invalidation.
- Selective reads and updates.
- Subscription to deterministic state transitions.
- Version, revision, provenance, expiration, and refresh metadata.
- Abstract persistence through `KnowledgeSessionPersistence`.

The VS Code adapter currently persists this state in `workspaceState` through [`VscodeKnowledgePersistence`](../src/extension/platform/vscode/vscodeKnowledgePersistence.ts). Neither the store nor its consumers assume that persistence is file based. A database, remote service, or MCP-backed persistence adapter can replace or complement it without changing reducers or consumers.

## Runtime Knowledge Session

[`RuntimeKnowledgeSession`](../src/extension/platform/knowledge/runtimeKnowledgeSession.ts) owns information that arrives or changes while the platform is operating. This includes MCP and agent results, LLM or RAG output, messages, skill/tool results, user contributions, network results, temporary derivations, and execution state.

It uses a separate normalized Redux Toolkit store and supports:

- Creation and initialization.
- Provider-backed asynchronous resolution and refresh.
- Add/update (`put`), query, typed listing, and subscription.
- Invalidation and removal.
- Disposal that clears only runtime-owned state.

The runtime session can read persistent resources through a read-through boundary. It does not copy them into the runtime store. Runtime invalidation or disposal therefore cannot corrupt persistent knowledge. A late provider response is discarded after disposal rather than repopulating a closed session.

The current Agile Delivery runtime adapter stores live records, assessments, reports, and cross-domain binding evidence in this generic runtime session. Those concrete resource types are adapter concerns, not separate platform state containers.

## Redux state and effects

[`sessionStore.ts`](../src/extension/platform/knowledge/sessionStore.ts) defines the normalized state, pure slice reducers, actions, and memoized selectors shared by both session kinds.

Reducers only apply deterministic transitions:

- Lifecycle changes.
- Load start/success/failure.
- Resource upsert, invalidation, and removal.
- Hydration and disposal.

External work is kept in an effect layer:

- [`KnowledgeSessionEngine`](../src/extension/platform/knowledge/sessionEngine.ts) coordinates cache lookup, provider calls, request status, and concurrent-load deduplication.
- [`KnowledgeProviderRegistry`](../src/extension/platform/knowledge/providerRegistry.ts) performs source selection and failure isolation.
- Provider adapters perform file, MCP, agent, network, memory, or other source I/O.
- Persistence adapters perform durable state I/O.

Concurrent requests for the same resource share one in-flight promise. The store always reaches a terminal ready/error state, and provider failures and attempts remain queryable.

## Provider contract

A provider implements one small contract:

```ts
interface KnowledgeProvider {
  readonly id: string;
  readonly kind: string;
  readonly tier: "workspace-override" | "authoritative" | "fallback";
  readonly priority?: number;
  canResolve(request: KnowledgeResourceRequest): boolean;
  resolve(request: KnowledgeResourceRequest, context: KnowledgeProviderContext):
    Promise<KnowledgeProviderValue | undefined>;
  discover?(request: KnowledgeDiscoveryRequest, context: KnowledgeProviderContext):
    Promise<KnowledgeProviderCandidate[]>;
}
```

To add an MCP, agent, RAG, messaging, skill, or network source:

1. Implement the provider contract and translate the external result into a typed knowledge value with provenance and refresh metadata.
2. Register it as an authoritative persistent or runtime provider with `KnowledgePlatform`.
3. Emit or invoke invalidation when the external source changes.

No session reducer, selector, canvas, Explorer, Details view, or composition consumer needs source-specific changes. [`MemoryKnowledgeProvider`](../src/extension/platform/knowledge/memoryKnowledgeProvider.ts) is the small in-memory reference implementation used by tests and can also back an authoritative session feed.

## Composition and consumer integration

[`KnowledgePlatform`](../src/extension/platform/knowledge/knowledgePlatform.ts) is created once during extension activation and shared with all consumers. It owns provider registries and both knowledge sessions.

[`CompositionLoader`](../src/extension/features/bridge/compositionLoader.ts) requests the unified worldview, domain worldviews, domain records, and cross-domain link sets by typed canonical identity. Registered domain connectors validate and normalize those values into a neutral `BridgeComposition`.

The composition is a read model, not a source of truth. The same composition feeds:

- Unified and focused domain canvas projections.
- FlowVerse Explorer and domain legend.
- FlowVerse Details.
- Canvas and diagram tools.
- Launch Intelligence context.

Workspace file creation/change/deletion invalidates persistent knowledge and regenerates the relevant projection. Runtime updates emit the same change path. The webview never reads source files.

## Canvas session

`FlowVerseCanvasSession` remains the UI and command lifecycle for one open generic canvas panel. System Architecture, Agile Delivery, and combined worldviews are perspectives in that session rather than separate knowledge sessions.

A perspective key is derived from its domain and view layers, for example:

```text
system-architecture:factory-landscape
agile-delivery:delivery-hierarchy
system-architecture:factory-landscape|agile-delivery:delivery-hierarchy
```

Perspective state includes selection, viewport, expanded cards, edge style, minimap, bundling, and data-flow controls. Switching perspective reprojects the same normalized composition, cancels stale commands, and restores valid perspective state. Closing the panel disposes only UI state; it does not clear the knowledge sessions or Explorer.

Manual layout pins remain VS Code workspace state keyed by worldview and perspective. They are view preferences, not domain knowledge.

## Launch and AI session

`LaunchSessionController` remains separate from both knowledge stores and the canvas session. It owns interaction-specific state such as the active selection, delivery scope, analysis artifacts, loading, and errors. It resolves its source context through the same provider-backed composition path.

Generated analysis is not promoted to authoritative knowledge implicitly. A future governed command/provider adapter must validate and write an approved result to its authoritative source or publish it as an explicitly classified runtime contribution.

## Lifecycle and ownership rules

- Persistent resources are owned only by the Persistent Knowledge Session.
- Runtime resources are owned only by the Runtime Knowledge Session.
- Runtime reads of persistent knowledge do not create duplicate runtime resources.
- Compositions and projections are disposable derived read models.
- Canvas and Launch sessions own interaction state, not canonical domain records.
- Runtime disposal leaves persistent knowledge intact.
- Extension deactivation flushes Launch persistence and awaits Persistent Knowledge Session persistence.
- No legacy session migration or parallel file-loading path is retained.

## Verification coverage

Automated coverage verifies:

- Workspace override > authoritative > fallback precedence.
- Authoritative resolution and fallback after provider failure.
- Failure attempts and terminal request errors.
- Concurrent-request deduplication.
- Runtime add/update/query/invalidate/remove/dispose behavior.
- Persistent reuse across runtime sessions and persistence hydration.
- Late-response safety after runtime disposal.
- Registration of a new authoritative memory provider without consumer changes.
- End-to-end loading of System Architecture and Agile Delivery demo canvases through live webviews.

The main checks are:

```bash
npm test
npm run typecheck:extension
npm run typecheck:webview
npm run build
npm run test:extension-host
npm run build:prod
```

## Deliberate follow-up work

- Concrete MCP, agent, RAG, messaging, skill, and network providers are not implemented yet; the provider and runtime contracts are ready for them.
- Retry, backoff, provider health, authentication renewal, and connection diagnostics belong in each external provider or a later provider-operations layer.
- Providers are responsible for declaring meaningful expiration or refresh metadata. Workspace change events and explicit refresh currently drive local invalidation.
- Discovery queries return the highest non-empty source tier; resolved resources are then cached by canonical identity.
- Runtime knowledge is intentionally ephemeral unless its authoritative provider can rehydrate it or a future runtime-persistence policy explicitly retains it.
