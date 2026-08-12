# Smart factory operations development data

This directory is the complete Smart Factory demonstration dataset. It composes the System Architecture and Agile Delivery domains, plus source-backed cross-domain links, through `unified-worldview.instance.json`.

With a released VSIX installed, open this directory as a folder in VS Code so FlowVerse discovers the connected example automatically. Use **FlowVerse: Open System Architecture Canvas**, **FlowVerse: Open Agile Delivery Canvas**, or **FlowVerse: Open Unified Worldview Canvas** to inspect each projection.

For extension development, select **Run FlowVerse with Smart Factory Example** in VS Code's Run and Debug view and press `F5`. The Extension Development Host opens this directory directly.

The architecture data covers industrial OPC UA and MQTT ingress, Kafka event streaming, eleven in-house Kubernetes microservices, operational data services, and five external vendor simulators. The Agile Delivery portfolio contains 66 epics, stories, tasks, bugs, and spikes—including 27 execution tasks—across three independent programme trees and up to five hierarchy levels. One deliberate cross-program dependency demonstrates coordination without turning the portfolio into a dependency web. Cross-domain links connect selected delivery evidence to the exact systems and modules it scopes or implements without copying either domain's records.

All organizations, teams, issue keys, endpoints, and operational content are fictional development fixtures. This directory can be removed without affecting packaged definitions and schemas under `resources/knowledge`.
