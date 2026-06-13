# Lab: Hunt for Threats Using the Microsoft Sentinel MCP Server — with Claude as the AI Agent

A hands-on lab connecting **Claude** (Claude Code or Claude Desktop) to the hosted **Microsoft Sentinel MCP server** and running natural-language threat hunts over the Sentinel data lake: discover tables, generate and execute KQL through MCP tools, and work the hypothesis → query → analyze → refine loop with an AI agent.

**Estimated time**: 45–60 minutes
**Builds on**: the [Sentinel Detection Flow lab](../sentinel-detection-flow/sentinel-detection-flow-lab.md) — reusing its workspace and `AzureActivity` data gives you something real to hunt.

> **Exam note**: The Sentinel MCP server, however, is a standard MCP endpoint with Entra ID OAuth, so any MCP-capable client can connect. This lab uses Claude to demonstrate exactly that point: the protocol is the interface, not the specific copilot.

## Architecture

```
Claude (Claude Code / Desktop)  ──MCP over HTTPS + Entra ID OAuth──►  Sentinel MCP server
        │                                                                    │ exposes tools
        │ natural-language prompts                                           ▼
        └────────────────────────────────────────────────►  search_tables / query_lake
                                                                             │
                                                                             ▼
                                                            Sentinel data lake (tenant-scoped,
                                                            all workspaces, mirrored tables)
```

## Prerequisites

- A Microsoft Sentinel workspace **onboarded to the Sentinel data lake** (Exercise 0 verifies/sets this up).
- An Entra ID account with permissions to read the data lake tables you'll hunt over — queries run under **your** identity.
- **Claude Code** (CLI) or **Claude Desktop** on a paid Claude plan (custom connectors / MCP support).
- Data in the workspace — the detection-flow lab's `AzureActivity` ingestion is ideal.

---

## Exercise 0 — Verify data lake onboarding (~10 min)

1. Open the **Microsoft Defender portal** → **Microsoft Sentinel → Data lake** (or Settings → Microsoft Sentinel → data lake settings, depending on rollout).
2. If the data lake isn't enabled, complete onboarding for your tenant and confirm your lab workspace is included. Mirroring of Analytics-tier tables into the lake begins automatically after onboarding.
3. Note the delay: freshly onboarded lakes need some time before tables are queryable.

**Checkpoint**: the data lake exploration experience lists your workspace's tables, including `AzureActivity`.

## Exercise 1 — Connect Claude to the Sentinel MCP server (~10 min)

The data-exploration collection endpoint: `https://sentinel.microsoft.com/mcp/data-exploration`

**Option A — Claude Code (CLI)**

```bash
claude mcp add --transport http sentinel-hunting https://sentinel.microsoft.com/mcp/data-exploration
claude
# inside the session:
/mcp
```

`/mcp` lists configured servers; select `sentinel-hunting` and follow the **Authenticate** flow — a browser window opens for **Entra ID sign-in**. Use the account that has lake permissions.

**Option B — Claude Desktop / claude.ai**

1. **Settings → Connectors → Add custom connector**.
2. Name: `Sentinel hunting`; URL: `https://sentinel.microsoft.com/mcp/data-exploration` → Add.
3. Complete the OAuth prompt with your Entra ID account, then enable the connector in a new chat.

**Checkpoint**: ask Claude — *"List the tools available from the Sentinel MCP server."* Expect data-exploration tools such as `search_tables` and `query_lake`.

**Troubleshooting**: auth loop or 401 → wrong account at the Entra prompt, or the account lacks lake read permissions. Tools listed but queries fail → workspace not onboarded to the lake (Exercise 0), or the table hasn't mirrored yet.

## Exercise 2 — First guided hunt: table discovery (~10 min)

The point of the MCP tools is that you stop memorizing schemas. Prompt Claude:

> *"Using the Sentinel MCP tools, find which tables in my data lake contain Azure management-plane activity, and describe their key columns."*

Claude should call `search_tables` and come back with `AzureActivity` (and possibly others). Follow up:

> *"Query the last 7 days of AzureActivity for resource group creation operations. Group by caller and show the count, the resource group names, and the caller IPs."*

Claude generates KQL and executes it via `query_lake`. If you ran the detection-flow lab, your `soc-lab-alert-rg` creations appear in the results — your own fingerprints, now hunted via an AI agent.

**Checkpoint**: results returned in-chat include your `soc-lab-alert` resource group creations with your account as `Caller`.

## Exercise 3 — The hunting loop, end to end (~15 min)

Work a full hypothesis-driven hunt, letting Claude handle the KQL while you steer:

1. **Hypothesis**: *"An unauthorized identity may be creating or deleting Azure resources outside business hours."*
2. **Query** — prompt: *"Test this hypothesis: query AzureActivity in the lake for write and delete operations in the last 14 days, bucket them by hour of day, and flag callers active outside 07:00–19:00 UTC."*
3. **Analyze** — interrogate the results: *"Which of these callers is the most anomalous and why? Show the evidence."*
4. **Refine** — narrow: *"Focus on [caller]. List every distinct operation they performed, ordered chronologically, and tell me if the sequence resembles persistence or cleanup behavior."*
5. **Escalate or close** — have Claude write the artifact: *"Draft a Sentinel scheduled analytics rule (KQL + suggested entity mappings) that would detect this pattern going forward."*

Step 5 closes the loop back to the [detection flow lab](../sentinel-detection-flow/sentinel-detection-flow-lab.md): paste the generated KQL into a new analytics rule and you've gone hunt → detection, which is exactly the workflow the exam's hunting domain describes (capturing hunt results as detections).

**Checkpoint**: you have a draft detection rule produced from a hunt — review the KQL critically before deploying; treat agent-generated KQL like any code review.

## Exercise 4 (optional) — Compare with the documented stack (~10 min)

If you have VS Code + GitHub Copilot available, add the same endpoint there (Command Palette → **MCP: Add Server** → HTTP → same URL) and rerun Exercise 2's first prompt. Observe that the tools, auth, and results are identical — only the client differs. This is the mental model the exam wants: *the MCP server exposes Sentinel's data-platform tools to any agentic client*.

## Exercise 5 — Cleanup

- Claude Code: `claude mcp remove sentinel-hunting`
- Claude Desktop: Settings → Connectors → remove the custom connector.
- If you're done with the whole lab series, follow the detection-flow lab's Exercise 7 to tear down the workspace.

---

## What you exercised, mapped to the SC-200 outline

| Lab step | Exam skill |
|---|---|
| Exercises 1–2 | Hunt for threats by using Notebooks / agentic clients, including connection to the Sentinel MCP Server |
| Exercises 2–3 | Hunt for threats by using KQL over data lake tables |
| Exercise 3 step 5 | Capture hunting results as custom detections |
| Exercise 0 | Sentinel data lake configuration awareness |

## Security considerations

- All MCP tool calls run under **your** Entra ID identity and permissions — least privilege applies.
- Hunt output flows through the AI client; follow your organization's policy on sending security data to AI assistants.
- Review agent-generated KQL before operationalizing it as a detection.

## See also

- [Hunt for Threats Using the Sentinel MCP Server — concept guide and diagram](hunt-for-threats-using-the-sentinel-mcp-server.md)
- [What is Microsoft Sentinel's support for MCP?](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-mcp-overview)
- [Microsoft Sentinel data lake documentation](https://learn.microsoft.com/en-us/azure/sentinel/datalake/)
