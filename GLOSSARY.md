# Glossary — Microsoft Defender & Microsoft Sentinel (SC-200)

A quick-reference dictionary of the key terms across Microsoft Sentinel and the Defender products, organized by area. For deeper write-ups on many of these, see the linked guides in the [README](README.md).

## Microsoft Sentinel

**Analytics rule** — the detection mechanism; runs query/logic to produce alerts and incidents. Types: Scheduled, NRT, Microsoft security (incident creation), Fusion, ML/Anomaly, Threat intelligence. See [Sentinel Detection Flow](sentinel-detection-flow/sentinel-detection-flow.md).

**Scheduled rule** — you author the KQL; runs on a schedule (5-min minimum); supports entity mapping. The workhorse.

**NRT (near-real-time) rule** — ~1-minute latency, runs every minute over that minute's data; single table only, restricted KQL; supports entity mapping.

**Fusion** — Microsoft ML correlation that combines low-fidelity signals across products into high-severity multistage-attack incidents. Black box, no entity mapping, enabled by default.

**Microsoft incident creation rule** — legacy rule that turns a Microsoft product's alerts into Sentinel incidents (filter by product/severity/name). No KQL, no entity mapping.

**Alert** — a single detection result emitted by a rule.

**Incident** — a container grouping related alerts; the unit analysts triage.

**Entity** — a recognized object in an alert (Account, Host, IP, File, URL, etc.).

**Entity mapping** — mapping KQL output columns to entity identifiers; only in scheduled + NRT rules. Powers entity pages, graph, UEBA, alert grouping, automation. See [Entity Mapping](entity-mapping/entity-mapping-in-analytics-rules.md).

**Automation rule** — orchestration: trigger + conditions + ordered actions (assign, tag, change status/severity, run playbook). The "react" layer.

**Playbook** — a Logic App that performs automated response (enrichment, isolation, notifications), often called by an automation rule.

**Workbook** — interactive visualization dashboard built on Azure Monitor workbooks; runs KQL to render charts/grids. Visualizes, doesn't detect. See [Sentinel Workbooks](sentinel-workbooks/sentinel-workbooks-visualize-and-monitor.md).

**Hunting query** — saved KQL for proactive, hypothesis-driven investigation (bookmarks, livestream).

**Notebook** — Jupyter-based advanced/programmatic analysis (Python + KQL), e.g., over the data lake or via the MCP server.

**Bookmark** — a saved finding from hunting you can promote to an incident.

**Livestream** — run a hunting query continuously to watch for matches in real time.

**Data connector** — ingests a data source into the workspace (Windows Security Events via AMA, Syslog/CEF, Microsoft Entra ID, Defender XDR, etc.). See [Entra ID connector tables](entra-id-connector-tables/entra-id-connector-tables.md).

**Content Hub** — marketplace of packaged solutions (connectors + rules + workbooks + playbooks) for standardized onboarding.

**Watchlist** — an imported reference list (VIP users, allowed IPs) you join to in queries.

**UEBA** — User and Entity Behavior Analytics; baselines behavior, surfaces anomalies and investigation priority scores (`BehaviorAnalytics` table).

**Analytics tier** — hot, interactive, queryable storage where rules run.

**Data lake tier** — low-cost, long-retention (up to 12 years) storage; not directly interactive — accessed via KQL jobs, summary rules, notebooks. See [Summary Rules, Workspace Tiers and Entity Tables](summary-rules-workspace-tiers/summary-rules-workspace-tiers-and-entity-tables.md).

**KQL job** — on-demand, retrospective query over data lake data; results land in an Analytics-tier table.

**Summary rule** — scheduled aggregation that writes results to an Analytics-tier summary table (cheap raw in lake, queryable aggregates).

**Search job** — async search over large/archived data; single table, ≤1 year, ≤1M results; output to a `_SRCH` table (Saved Searches).

**SOC optimization** — recommendations for data value (unused tables) and threat-based coverage gaps.

**Sentinel Graph** — entity-relationship/blast-radius visualization built from entity data.

**Sentinel MCP server** — hosted Model Context Protocol endpoint (Entra ID auth) exposing data-lake tools (`search_tables`, `query_lake`) for agentic hunting from clients like VS Code/Copilot. See [Hunt with the Sentinel MCP Server](sentinel-mcp-threat-hunting/hunt-for-threats-using-the-sentinel-mcp-server.md).

**Workspace Manager** — central management of content across multiple Sentinel workspaces.

**Azure Lighthouse** — cross-subscription/cross-tenant delegated management (multi-workspace Sentinel).

## Defender XDR (portal-wide)

**Microsoft Defender portal** — security.microsoft.com; unified home for Endpoint, Office 365, Identity, Cloud Apps, and Sentinel.

**Incident (XDR)** — auto-correlated set of alerts across Defender products.

**Action center** — log of remediation actions; **Pending** (approve/reject) and **History** (revert completed actions).

**Automated investigation and response (AIR)** — Defender investigates alerts autonomously and proposes/takes remediation.

**Attack disruption** — automatic, high-confidence containment during active attacks (isolate device, disable account); configured in Defender XDR settings.

**Advanced hunting** — cross-product KQL hunting over the `Device*`, `Email*`, `Identity*` tables.

**Custom detection rule** — your Advanced Hunting query turned into a recurring detection (needs Timestamp + ReportId + an entity column).

**Suppression rule** — hides/auto-resolves false-positive alerts; scope to device group (not "any device") to preserve posture.

**Embedded Copilot / Security Copilot** — AI assistance (incident summaries, guided responses); needs SCU capacity + the Defender XDR plugin. See [Security Copilot Defender XDR plugin](security-copilot-defender-xdr-plugin/security-copilot-defender-xdr-plugin.md).

## Defender for Endpoint (MDE)

**Onboarding** — connecting a device's sensor (Intune / GPO / ConfigMgr / local script / VDI).

**Device group** — tag-based membership with a **rank** (1 = highest); used to scope actions/automation.

**Live response** — interactive remote shell from the portal (inspect/collect/run/remediate); works on isolated devices; toggles: Live response, for servers, unsigned scripts. See [Live Response](mde-live-response/live-response.md).

**Investigation package** — one-shot, non-interactive forensic ZIP (processes, connections, autoruns, logs); no memory dump. See [Collect Investigation Package](mde-investigation-package/collect-investigation-package.md).

**Deep analysis** — detonate a PE file in Microsoft's cloud sandbox for a behavioral report. See [Submit for Deep Analysis](mde-deep-analysis/submit-for-deep-analysis.md).

**Indicators (IoCs)** — block/allow/audit by file hash, IP, URL/domain, or certificate; URL/IP enforcement needs Custom network indicators on + network protection. See [Blocking Custom IPs & URLs](mde-blocking-custom-ips-urls/blocking-custom-ips-and-urls-with-defender-for-endpoint.md).

**ASR rules** — attack surface reduction behavioral rules (block executable content from email, block unsigned USB processes); need Defender AV primary; Audit/Block/Warn. See [ASR Rules](mde-asr-rules/attack-surface-reduction-rules.md).

**Network protection** — blocks outbound connections to malicious IPs/URLs (enforces non-Edge indicators).

**Tamper protection** — prevents changes to security settings; don't disable.

**EDR in block mode** — lets EDR remediate even when a third-party AV is primary.

**Web content filtering** — category-based web blocking (complements indicators).

**Device timeline** — per-device chronological event stream (events before/after an alert); flags, CSV, 6-month history. See [Device Timeline](mde-device-timeline/device-timeline-events-before-alert.md).

**Entity page (IP/device/user/file)** — pivot from alert evidence to an entity's full context. See [IP Address Entity Page](mde-ip-entity-page/ip-address-entity-page.md).

## Defender for Identity (MDI)

**Sensor** — installed on domain controllers (and AD FS/AD CS); reads local traffic/events; no port mirroring.

**Entity tags** — Sensitive, Honeytoken, Exchange server tags applied to AD entities.

**Honeytoken** — decoy account; any activity is high-signal.

**Sensitive groups report** — audit of all changes to privileged/tagged groups (`IdentityDirectoryEvents`). See [Modifications to Sensitive Groups Report](mdi-sensitive-groups-report/modifications-to-sensitive-groups-report.md).

**Lateral movement paths (LMPs)** — graph of how an attacker could reach a privileged account.

**Identity security posture** — hardening recommendations (unsecure Kerberos delegation → fix delegation on computer objects; legacy protocols → disable; unsecure LDAP → enforce LDAP signing; local admin password → deploy LAPS).

## Defender for Office 365 (MDO)

**Safe Attachments** — pre-delivery detonation of email attachments in a sandbox.

**Safe Links** — time-of-click URL rewriting/checking.

**ZAP (zero-hour auto purge)** — post-delivery removal of mail later found malicious (to Junk/Quarantine).

**Threat Explorer / Explorer** — investigate and take action on email (soft/hard delete) across mailboxes.

**Quarantine** — holding area for blocked messages; reviewed/released on the Review page.

## Defender for Cloud (formerly Azure Security Center)

**CSPM** — posture management; **Secure Score**; **recommendations** (with Quick Fix).

**Defender plans** — per-resource-type workload protection (Servers, Storage, SQL, Key Vault, Containers...). Enabling = a Contributor/Owner action, not Security Admin.

**Workflow automation** — triggers Logic Apps from recommendations/alerts; where you test remediation playbooks.

**Continuous export** — stream alerts/recommendations to Log Analytics or **Event Hubs** (Event Hubs for third-party SIEM).

**Data collection / auto-provisioning** — agent + security-event tier (All/Common/Minimal) to collect events.

**Cloud connectors** — onboard **AWS/GCP**; pair with **Azure Arc** for servers.

**Regulatory compliance** — posture vs. standards (Microsoft Cloud Security Benchmark, PCI-DSS, ISO).

**Email notifications** — configured under **Pricing & settings** (per subscription).

**Key Vault firewall** — network access control for a Key Vault (selected networks / IPs / private endpoint); the *where-from* control, separate from RBAC (*who*).

## Ingestion & infrastructure

**AMA (Azure Monitor Agent)** — current collection agent; replaces legacy MMA/Log Analytics agent.

**DCR (data collection rule)** — defines what to collect (XPath filter), transform, and where to send. **DCR association** activates it on a resource.

**Azure Arc** — projects on-prem/other-cloud servers into Azure as resources so DCRs/AMA connectors can target them. See [Arc + AMA onboarding](arc-ama-onboarding/onboard-onprem-servers-arc-ama.md).

**ASIM** — Advanced Security Information Model; normalizing/unifying parsers (e.g., `_Im_Dns`) for source-agnostic queries.

## Common Advanced Hunting tables

| Table | Contents |
|---|---|
| `DeviceProcessEvents` | Process creation |
| `DeviceNetworkEvents` | Network connections |
| `DeviceFileEvents` | File create/modify/delete |
| `DeviceRegistryEvents` | Registry operations |
| `DeviceLogonEvents` | Device logons |
| `DeviceEvents` | ASR, AMSI, USB, misc (catch-all) |
| `EmailEvents` / `EmailAttachmentInfo` / `EmailUrlInfo` | Email delivery, attachments, URLs |
| `IdentityLogonEvents` | Identity authentications |
| `IdentityQueryEvents` | LDAP/SAMR/DNS recon queries |
| `IdentityDirectoryEvents` | Directory changes (group membership, etc.) |
| `AlertEvidence` / `AlertInfo` | Alert entities and metadata |
| `SigninLogs` / `AuditLogs` | Entra ID sign-in / audit activity |
| `SecurityAlert` | Security-product alerts |
| `SecurityEvent` | Windows security events (4624, 4720, etc.) |
| `BehaviorAnalytics` | UEBA |

## Useful event IDs

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4688 | Process creation |
| 4720 | User account created |
| 4728 / 4732 / 4756 | Member added to (global / local / universal) security group |
