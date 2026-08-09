# FlowVerse Launch Intelligence

## Summary

Evolve the plugin into an ontology-backed launch intelligence layer connecting:

**Raw intent → requirements → Jira → solution decisions → architecture impact → delivery plan → verification → release evidence**

The first production wedge will be **intent-to-delivery**. It stops at an explicitly approved Jira plan; source-code execution and PR creation remain later capabilities.

Three product promises govern every feature:

- **Launch faster:** reduce discovery, planning, and coordination time.
- **Break nothing:** reveal downstream impact, compatibility risks, blockers, and missing tests.
- **Know exactly why:** every conclusion carries evidence, provenance, confidence, freshness, and decision history.

## Ten-Feature Portfolio

### 1. Launch Brief Compiler — P0

Turn raw specifications, meeting notes, or incomplete epics into a reviewable launch brief.

- Extract objectives, actors, functional requirements, NFRs, constraints, risks, assumptions, acceptance criteria, and open questions.
- Classify each item through Core Ontology and Agile Delivery concepts.
- Suggest affected capabilities, systems, modules, and data flows.
- Show a requirement-to-architecture coverage map.
- Generate an approval-gated epic/story/task proposal.
- Success: every requirement is traced to architecture and acceptance evidence or shown as an explicit gap.

### 2. Solution Scenario Studio — P0

Add an **Agent-proposed solutions** section to the Jira report.

- Present two or three credible options with embedded mini architecture diagrams.
- Show affected systems, modules, APIs, messages, flows, risks, blockers, assumptions, delivery range, evidence, and confidence.
- Compare options side by side using consistent criteria.
- **Explore architecture** opens the detailed canvas focused on the selected proposal.
- **Adopt as plan** records an approved decision and feeds the delivery planner.
- Proposed topology remains an overlay; it never silently changes published architecture.

### 3. Change Impact Atlas — P0

Provide a complete, explainable blast-radius view for a Jira change.

- Highlight direct and propagated impact through APIs, messages, files, databases, infrastructure, interactions, and data flows.
- Attach Jira cards to exact systems, modules, interactions, endpoints, risks, or flows.
- Distinguish `direct`, `propagated`, `potential`, and `unresolved` impact.
- Clicking an impact path explains the relationship, evidence, confidence, owning team, and affected work.
- Traverse only according to unified-worldview relationship propagation semantics, with cycle and depth limits.

### 4. Architecture-Aware Delivery Planner — P0

Convert an approved solution into executable delivery work.

- Decompose work by system, module, contract, and owning discipline.
- Produce an ordered implementation plan, dependency diagram, verification obligations, rollback considerations, and Jira hierarchy.
- Preserve exact architecture IDs in every generated work package.
- Estimates are evidence-based ranges using Jira history and architecture complexity; insufficient evidence produces `unknown`, not invented precision.
- Creation uses the existing visible Jira plan and one-use approval-token flow.

### 5. Dependency and Critical-Path Radar — P0

Combine Jira blockers with architecture prerequisites and cross-team dependencies.

- Show confirmed blockers separately from AI-detected potential blockers.
- Trace downstream systems, work items, contracts, and release gates affected by each blocker.
- Detect dependency cycles and missing prerequisite work.
- Separate topology-only critical chains from estimate-based critical paths.
- Let the agent draft missing dependency tickets or links, subject to approval.

### 6. Contract and Flow Compatibility Guardian — P1

Determine whether a proposed solution is compatible with existing consumers and flows.

- Compare current and proposed immutable architecture snapshots.
- Classify API, message, file, protocol, and data-flow changes as additive, compatible, conditionally compatible, breaking, or unknown.
- Show before/after diagrams and affected producers, consumers, and downstream flows.
- Generate versioning, migration, coordination, rollback, and test obligations.
- Breaking or unknown high-criticality changes create readiness gates.

### 7. Acceptance-to-Test Coverage Designer — P1

Turn requirements and architecture paths into a traceable verification strategy.

- Map acceptance criteria and NFRs to modules, interactions, data flows, and architecture risks.
- Propose unit, component, contract, integration, resilience, performance, security, and end-to-end tests.
- Render a coverage matrix and color impacted paths as covered, partial, missing, or blocked.
- Distinguish proposed tests from observed test evidence.
- Require every high-criticality impacted flow to have evidence or an approved exception.

### 8. Intent-to-Implementation Drift Sentinel — P1

Detect divergence between approved intent, delivery scope, architecture, and workspace changes.

- Compare the selected solution and Jira plan with architecture revisions, changed workspace files, test files, and validation results.
- Identify unplanned systems touched, planned modules untouched, stale links, missing architecture updates, and insufficient tests.
- Render planned impact versus observed impact.
- Use a read-only repository evidence adapter; direct filesystem inspection overrides retrieval guesses.
- Findings remain advisory until a team explicitly enables policy enforcement.

### 9. Release Confidence Command Center — P2

Produce a defensible release recommendation.

- Aggregate Jira completion, blockers, architecture risks, compatibility, test evidence, approvals, drift, freshness, and unresolved decisions.
- Show a release heatmap over the architecture diagram.
- Return `Ready`, `Needs review`, or `Not ready`; do not use an opaque AI score.
- Treat missing evidence as `unknown`, distinct from failure.
- Every failed or unknown gate links to its evidence, owner, Jira work, and architecture element.
- Overrides require an actor, rationale, expiry, and decision record.

### 10. Living Product Decision Memory — P2

Preserve why products and systems evolved as they did.

- Store approved solutions, rejected alternatives, architecture decisions, risk acceptances, waivers, and release outcomes as versioned decision records.
- Link decisions bidirectionally to Jira work, architecture entities, evidence, and successor decisions.
- Let users ask “Why is this designed this way?” from any Jira or architecture card.
- Clearly distinguish current facts, superseded decisions, and rejected proposals.
- Model-generated records remain proposed until explicitly approved.

## Platform and Interface Changes

### Knowledge resolution

Package the architecture JSON, Core Ontology, domain ontologies, worldview instances, unified worldview, and link sets with the extension as a validated, read-only knowledge pack.

Resolution order:

1. Explicit `flowverse.worldviewPath`.
2. Valid workspace knowledge pack.
3. Bundled plugin knowledge pack.

This preserves existing workspace workflows while enabling zero-configuration production use.

### Cross-domain analysis layer

Add an extension-host semantic layer separate from React Flow:

- `CrossDomainContext`: Jira graph, unified composition, architecture index, decisions, and source revisions.
- `ArchitectureKnowledgeIndex`: indexed systems, modules, interactions, endpoints, flows, risks, review questions, incoming/outgoing relationships, and ownership.
- `AnalysisArtifact<T>`: versioned feature result containing scope, claims, evidence, issues, source revisions, and generator version.
- `DerivedClaim<T>` states: `observed`, `derived`, `proposed`, `approved`, `rejected`, or `superseded`.
- Every claim has deterministic confidence, rationale, and evidence references.

Do not add analysis fields to canonical `worldview.instance.json` or `*.architecture.json`.

### Agent tools

Keep all seven existing Jira LM tools and their inputs unchanged. Add:

- `flowverse_analyse`: read-only dispatcher with typed feature inputs for the ten analyzers.
- `flowverse_plan_changes`: produces a digest-bound, previewable Jira or workspace change plan.
- `flowverse_apply_changes`: applies only an approved, unexpired plan after revision preconditions pass.

The agent supplies judgment and prose. Deterministic code owns retrieval, graph traversal, validation, confidence policy, exact ID binding, readiness gates, and write authorization.

### Webview experience

- Keep the Jira report as the concise entry surface.
- Add a launch-intelligence summary containing solution cards, mini impact canvas, key risks, plan stages, and readiness.
- Add a dedicated detailed editor with `Solutions`, `Impact`, `Plan`, `Verification`, and `Readiness` views.
- Use an **Evidence Spine** connecting requirement → decision → architecture impact → work → test → release evidence.
- Reuse the generic FlowVerse canvas projection and domain presentation registries; render analysis and Jira cards without cloning canonical data.
- Extend architecture bridge entities to expose interactions, endpoints, data flows, risks, and review questions.
- Route report focus actions through the versioned `flowverse.diagram.execute` command contract.
- Fix plugin support for unified-worldview `backward` propagation instead of silently treating it as `forward`.

## Production Guardrails

- Read-only analysis is the default.
- AI output never becomes canonical architecture or Jira state without validation and explicit approval.
- Draft artifacts live in extension workspace storage; exporting tracked analysis or decision files requires review.
- Jira writes verify issue revision/version before applying.
- Architecture proposals verify file hashes and pass strict schema and semantic validation.
- Mixed Jira and filesystem writes execute as a visible, resumable saga rather than pretending to be transactional.
- No fallback from live Jira to synthetic data.
- No secrets, customer data, or raw credentials enter prompts, artifacts, logs, or diagrams.
- Cache keys include feature, normalized input, and source-revision fingerprints.
- Changed source revisions mark dependent artifacts stale.
- Connector loads are bounded and parallel; traversals are cycle-safe and capped.
- One malformed system produces a partial-data warning without disabling unrelated features.

## Delivery Sequence

1. **Foundation:** bundled knowledge resolver, strict validation, architecture knowledge index, analysis artifact schema, source revisions, confidence policy, and propagation compatibility.
2. **Launch cockpit:** Launch Brief Compiler, Solution Scenario Studio, and Change Impact Atlas.
3. **Delivery intelligence:** Delivery Planner and Critical-Path Radar.
4. **Safety intelligence:** Compatibility Guardian, Test Coverage Designer, and Drift Sentinel.
5. **Lifecycle governance:** Release Confidence and Decision Memory.

Each phase is additive and keeps the current Jira report, raw JSON report, service preview, layered view, and existing LM tools operational.

## Test Plan

- Contract tests for bundled/workspace resolution, artifact schemas, exact entity references, source revisions, and old message compatibility.
- Semantic tests for forward, backward, bidirectional, and non-propagating relationships.
- Traversal tests for cycles, caps, duplicate paths, missing modules, stale links, disabled systems, and partial source failures.
- Security tests for prompt injection in Jira content, unsafe URLs, secret redaction, workspace trust, and simulator/live isolation.
- Feature tests for all ten analyzers using synthetic Jira and architecture fixtures.
- Approval tests for expired tokens, changed Jira revisions, changed file hashes, partial failures, retries, and compensation instructions.
- UI tests for embedded diagrams, focus navigation, keyboard access, reduced motion, evidence drawers, empty states, stale states, and failure isolation.
- Regression gates: existing plugin tests, extension/webview typechecks, production build, strict architecture validation, unified-worldview validation, and schema parity.
- Performance acceptance: overlay updates must not relayout the base graph; repeated analysis must reuse revision-indexed snapshots; cancellation must stop Jira fetches and long traversals.

## Assumptions and Defaults

- The launch wedge is **intent-to-delivery**, based on the stated product vision.
- The first production release stops at an approved Jira delivery plan; source editing and PR creation are later opt-in capabilities.
- Estimates are ranges with assumptions and confidence, never commitments.
- Published architecture JSON remains authoritative and read-only from analysis views.
- Decision Memory begins as validated decision artifacts; it does not introduce a new ontology domain until the contract stabilizes.
- Repository and test evidence use local read-only adapters first; GitHub, CI, deployment, and observability connectors remain additive.
- Implementation execution will create the required workspace evidence and service-manual artifacts; this planning pass remains read-only.
