# Domain Integration

This guide describes how a domain participates in FlowVerse, how source records are resolved, and where domain-specific behavior belongs.

## Domain Extension Model

Each supported domain is a package under `src/domains/` with two registered halves:

- an extension-host module containing a connector and validation adapter;
- a renderer module containing canvas presentation, explorer behavior, details, and visual themes.

`AbstractDomainConnector` owns the shared setup: resolve the domain worldview, keep file access inside the workspace boundary, parse it, invoke validation, and pass the validated registry to the domain connector. A connector normalizes domain records into the neutral bridge model; its validation adapter validates the domain worldview and extracts registered instance IDs.

The generic composition loader, explorer, canvas, legend, and details shell do not contain Agile Delivery or System Architecture special cases. A new domain implements the contracts and is added to the extension and renderer registries.

Current domain packages:

```text
src/domains/
  systemArchitecture/
    extension/   Connector and validation adapter
    canvas/      Projection, layout, cards, edges, and themes
    details/     Architecture details module
  agileDelivery/
    extension/   Jira connector, validation adapter, and runtime source
    canvas/      Work-item presentation and themes
    details/     Agile Delivery details module
    model/       Canonical Jira runtime merge model
```

## Resources

The repository keeps product knowledge and disposable development data separate:

```text
resources/
  knowledge/     Packaged definitions, schemas, and the runtime Agile Delivery profile
  examples/
    unified-worldview.instance.json
    architecture/
    agile-delivery/
    cross-domain-links/
```

`resources/knowledge/` contains shared definitions, schemas, and the data-free Agile Delivery profile used by connected session sources. The extension-owned unified worldview is defined in `unifiedWorldviewModule.ts`; it is not discovered from workspace data. All demonstration systems, modules, Jira items, and cross-domain assertions live under `resources/examples/` and can be removed before production packaging without changing the product runtime.

The development example includes both current domains, 23 architecture boundaries, 24 multi-level Agile Delivery records, and explicit delivery-to-architecture link sets. See the [Smart Factory Operations Example](smart-factory-operations-example.md).

## Configuration and Data Resolution

The generic canvas starts from the extension-owned two-domain composition. System Architecture resolves from `architecture/worldview.instance.json` in the workspace. Agile Delivery records and cross-domain relationships are supplied by connected sources and retained only in the active runtime/session composition.

The single-service preview resolves its System Architecture worldview separately:

1. `flowverse.worldviewPath`;
2. the nearest `flowverse.authoring.json` and its `worldviewInstancePath`;
3. common workspace architecture paths;
4. the nearest `worldview.instance.json` found by walking up from the service file;
5. service-local preview vocabulary when no worldview is available.

Jira base URL resolution:

1. `FLOWVERSE_JIRA_BASE_URL` from the VS Code process;
2. the machine-local `flowverse.jiraBaseUrl` setting;
3. `dev.env` only in an Extension Development Host.

Use **FlowVerse: Set Jira Base URL** to configure an installed extension. Jira credentials come from VS Code SecretStorage or the process `JIRA_PAT`; never place credentials in an environment file or workspace file.
