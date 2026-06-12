# SOC

Notes, guides, and architecture diagrams for Microsoft Sentinel and Defender XDR security operations.

## Sentinel Detection Flow

A walkthrough of how Microsoft Sentinel turns raw signals into triageable incidents: an **analytics rule** runs a KQL query (or matches threat-intelligence indicators) against your workspace log tables, emits **alerts** when results are found, and groups those alerts into **incidents** for analysts. See [the guide](sentinel-detection-flow/sentinel-detection-flow.md) with architecture diagram.

## Hunt for Threats Using the Sentinel MCP Server

AI-assisted threat hunting over the Microsoft Sentinel data lake: connect **GitHub Copilot in agent mode** (VS Code) to the hosted **Sentinel MCP server** and hunt across tenant workspaces with natural language, notebooks, and the `search_tables` / `query_lake` tools. See [the guide](sentinel-mcp-threat-hunting/hunt-for-threats-using-the-sentinel-mcp-server.md) with architecture diagram.

## Summary Rules, Workspace Tiers and Entity Tables

How the Sentinel platform balances cost and queryability: the **Analytics** and **Data lake** tiers, **summary rules** for scheduled aggregation, **KQL jobs** for retrospective hunts, and the **entity tables** behind Sentinel Graph and UEBA. See [the guide](summary-rules-workspace-tiers/summary-rules-workspace-tiers-and-entity-tables.md) with architecture diagram.

## Blocking Custom IPs & URLs with Defender for Endpoint

Blocking specific IP addresses and URLs on devices with **custom network indicators**: the **Custom network indicators** advanced feature, the network protection (block mode) and Defender AV prerequisites, and enforcement via SmartScreen vs. network protection. See [the guide](mde-blocking-custom-ips-urls/blocking-custom-ips-and-urls-with-defender-for-endpoint.md) with architecture diagram.
