# SOC

Notes, guides, and architecture diagrams for Microsoft Sentinel and Defender XDR security operations.

📖 **[Glossary](GLOSSARY.md)** — quick-reference dictionary of Sentinel & Defender terms, key Advanced Hunting tables, and useful Windows event IDs.

## Sentinel Detection Flow

A walkthrough of how Microsoft Sentinel turns raw signals into triageable incidents: an **analytics rule** runs a KQL query (or matches threat-intelligence indicators) against your workspace log tables, emits **alerts** when results are found, and groups those alerts into **incidents** for analysts. See [the guide](sentinel-detection-flow/sentinel-detection-flow.md) with architecture diagram, plus a **[hands-on lab](sentinel-detection-flow/sentinel-detection-flow-lab.md)** that builds the entire flow — ingestion, KQL detection, scheduled rule with entity mapping, a self-triggered incident, and an automation rule — on the Sentinel free trial in ~90 minutes.

## Hunt for Threats Using the Sentinel MCP Server

AI-assisted threat hunting over the Microsoft Sentinel data lake: connect **GitHub Copilot in agent mode** (VS Code) to the hosted **Sentinel MCP server** and hunt across tenant workspaces with natural language, notebooks, and the `search_tables` / `query_lake` tools. See [the guide](sentinel-mcp-threat-hunting/hunt-for-threats-using-the-sentinel-mcp-server.md) with architecture diagram, plus a **[hands-on lab](sentinel-mcp-threat-hunting/sentinel-mcp-hunting-lab-claude.md)** that runs the full hunting loop with **Claude** as the MCP client (~45–60 min, builds on the detection-flow lab's data).

## Summary Rules, Workspace Tiers and Entity Tables

How the Sentinel platform balances cost and queryability: the **Analytics** and **Data lake** tiers, **summary rules** for scheduled aggregation, **KQL jobs** for retrospective hunts, and the **entity tables** behind Sentinel Graph and UEBA. See [the guide](summary-rules-workspace-tiers/summary-rules-workspace-tiers-and-entity-tables.md) with architecture diagram.

## Entity Mapping in Analytics Rules

How **entity mapping** enriches alerts and incidents by mapping KQL result columns to entity identifiers (Account, Host, IP, File/URL) — the building blocks for entity pages, the graph, UEBA, alert grouping, and automation. Covers why it's available **only in scheduled and NRT rules** and not in TI/Fusion/ML types. See [the guide](entity-mapping/entity-mapping-in-analytics-rules.md) with architecture diagram.

## Sentinel Workbooks: Visualize & Monitor

Visualizing and monitoring connected data with **workbooks** — custom, interactive dashboards built on Azure Monitor workbooks that run KQL against the workspace tables. Covers templates vs. saved instances, the visualize-vs-detect distinction, and how workbooks differ from hunting, notebooks, and overview pages. See [the guide](sentinel-workbooks/sentinel-workbooks-visualize-and-monitor.md) with architecture diagram.

## Entra ID Connectors and Their Sentinel Tables

Which table does Entra ID data land in? The **Microsoft Entra ID** connector routes sign-in and audit activity to `SigninLogs` and `AuditLogs`; the **Microsoft Entra ID Protection** connector routes identity risk alerts to `SecurityAlert`. Clears up the common "Entra ID connector writes to SecurityAlert" misconception, with the table-selection exam tells. See [the guide](entra-id-connector-tables/entra-id-connector-tables.md) with architecture diagram.

## Onboarding On-Prem Servers with Azure Arc + AMA

Why on-premises servers must be **Arc-enabled** before DCRs and the **Windows Security Events via AMA** connector can target them: install the Azure Connected Machine agent → the server becomes an Azure resource in the picker → the connector creates a DCR and deploys AMA → events flow to `SecurityEvent`. See [the guide](arc-ama-onboarding/onboard-onprem-servers-arc-ama.md) with architecture diagram.

## Blocking Custom IPs & URLs with Defender for Endpoint

Blocking specific IP addresses and URLs on devices with **custom network indicators**: the **Custom network indicators** advanced feature, the network protection (block mode) and Defender AV prerequisites, and enforcement via SmartScreen vs. network protection. See [the guide](mde-blocking-custom-ips-urls/blocking-custom-ips-and-urls-with-defender-for-endpoint.md) with architecture diagram.

## Attack Surface Reduction (ASR) Rules

Blocking executable content from email and unsigned processes from USB drives with **ASR rules**: the built-in rule catalog, the Defender Antivirus primary-AV prerequisite, Audit/Block/Warn rollout, deployment via Intune/GPO/PowerShell, and monitoring ASR events in `DeviceEvents`. See [the guide](mde-asr-rules/attack-surface-reduction-rules.md) with architecture diagram.

## Defender for Identity: Modifications to Sensitive Groups Report

A complete audit trail of every change to sensitive AD groups (built-in privileged groups + manually tagged accounts/groups), detected by the **MDI sensor** on domain controllers — including the report-vs-alert distinction and the `IdentityDirectoryEvents` hunting companion. See [the guide](mdi-sensitive-groups-report/modifications-to-sensitive-groups-report.md) with architecture diagram.

## Security Copilot: Defender XDR Plugin & Embedded Experience

Why the **Microsoft Defender XDR plugin** must be enabled in Security Copilot for the embedded Copilot features in the Defender portal to work — incident summaries, guided responses, script analysis, NL-to-KQL, and incident reports — plus the SCU capacity prerequisite and the standalone-vs-embedded distinction. See [the guide](security-copilot-defender-xdr-plugin/security-copilot-defender-xdr-plugin.md) with architecture diagram.

## Device Page Timeline Tab: Events Before an Alert

Reviewing the events that occurred **before** a high-severity alert using the **Timeline tab** on the device page: the per-device chronological event stream, filtering by MITRE technique, event flags, CSV export, 6-month history, and how the timeline differs from the alert story, Advanced Hunting, and the incident graph. See [the guide](mde-device-timeline/device-timeline-events-before-alert.md) with architecture diagram.

## Defender for Endpoint: Live Response

Remotely connecting to a device from the Defender portal with **live response**: the cloud session channel (works on isolated devices), the inspect/collect/run/remediate command families, the three Advanced features toggles (live response, for servers, unsigned scripts), basic vs. advanced RBAC, and how it differs from investigation packages and AIR. See [the guide](mde-live-response/live-response.md) with architecture diagram.

## Defender for Endpoint: Collect Investigation Package

Gathering forensic evidence with a **one-shot, non-interactive** action that minimizes user disruption: what the investigation package contains (and what it doesn't — no memory dump), the silent background collection flow, Action center download, and the package-vs-live-response-vs-AIR discrimination table. See [the guide](mde-investigation-package/collect-investigation-package.md) with architecture diagram.

## Defender for Endpoint: Submit for Deep Analysis

Detonating a suspicious file in **Microsoft's cloud sandbox** for a behavioral report — process activity, registry changes, network contacts, dropped files — with the online-device + sample-collection prerequisites, the PE-files-only scope, and how deep analysis differs from live response file commands and MDO Safe Attachments. See [the guide](mde-deep-analysis/submit-for-deep-analysis.md) with architecture diagram.

## From the Alert Process Tree to the IP Address Entity Page

Investigating an external IP's **related activity, alerts, and context** directly from incident evidence: the entity-pivot model, the entity page sections (TI overview, tenant-wide incidents, organizational prevalence), and the Add indicator / Go hunt pivot actions. See [the guide](mde-ip-entity-page/ip-address-entity-page.md) with architecture diagram.
