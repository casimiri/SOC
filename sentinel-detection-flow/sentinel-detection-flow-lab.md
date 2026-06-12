# Lab: Sentinel Detection Flow — From Raw Event to Incident to Automation

A hands-on lab that walks the full detection flow end to end: ingest data into Microsoft Sentinel, write a KQL detection, build a scheduled analytics rule with entity mapping, trigger it yourself, watch the alert become an incident, and attach an automation rule that triages it.

**Estimated time**: 60–90 minutes (includes ~15 min of waiting for ingestion/rule cycles)
**Cost**: minimal — Microsoft Sentinel is free for the first 31 days (up to 10 GB/day) on a new workspace, and the Azure Activity connector ingests for free.

## Lab design

The trigger is something you fully control: creating an Azure **resource group** whose name contains `soc-lab-alert`. The Azure Activity connector records the operation, your analytics rule detects it, and an incident is born — no licenses, no test malware, completely reproducible.

```
You create RG "soc-lab-alert-rg"
        │
        ▼
AzureActivity table  ──►  Scheduled analytics rule (KQL + entity mapping)
                                    │ results found
                                    ▼
                                  Alert ──► Incident ──► Automation rule (assign + tag + severity)
```

## Prerequisites

- An Azure subscription where you can create resource groups (Owner or Contributor on the subscription, plus the ability to assign the connector — Reader on the subscription is required for the Azure Activity data connection).
- Microsoft Sentinel Contributor (or higher) on the resource group that will host the workspace.

---

## Exercise 0 — Environment setup (~10 min)

1. Azure portal → **Resource groups → Create**: name `rg-sentinel-lab`, any region (e.g., West Europe).
2. **Log Analytics workspaces → Create**: resource group `rg-sentinel-lab`, name `law-sentinel-lab`, same region → Create.
3. Search **Microsoft Sentinel → Create**, select `law-sentinel-lab` → **Add**. The 31-day free trial banner should appear.

**Checkpoint**: Microsoft Sentinel opens on the Overview blade with your workspace name in the header.

## Exercise 1 — Ingest data: Azure Activity connector (~15 min, mostly waiting)

1. Sentinel → **Content management → Content hub** → search **Azure Activity** → Install.
2. **Configuration → Data connectors → Azure Activity → Open connector page**.
3. Under the configuration step, **Launch Azure Policy Assignment wizard**: scope = your subscription → Review + Create. (This deploys the diagnostic settings that route activity logs to your workspace.)
4. Ingestion is not instant — Azure Activity typically starts flowing in **10–15 minutes**.

While waiting, generate some activity: create and delete a throwaway resource group named `test-noise-rg` so the table has data.

**Checkpoint** — in **Logs**, run:

```kql
AzureActivity
| take 10
```

If rows return, ingestion is live. If empty after 20 minutes, check that the policy assignment succeeded and re-save the diagnostic setting on the subscription (Monitor → Activity log → Export activity logs).

## Exercise 2 — Write and test the detection KQL (~10 min)

In **Logs**, build the detection incrementally:

```kql
AzureActivity
| where OperationNameValue =~ "Microsoft.Resources/subscriptions/resourceGroups/write"
| where ActivityStatusValue in~ ("Success", "Succeeded")
| where ResourceGroup has "soc-lab-alert"
| project TimeGenerated, Caller, CallerIpAddress, ResourceGroup, OperationNameValue
```

Run it — it should return **nothing** yet (you haven't created the trigger RG). That's correct: you're testing the query compiles and the logic is sound. Remove the `ResourceGroup` filter temporarily and you should see your `test-noise-rg` creation from Exercise 1 — proof the detection logic works.

**Concepts to notice**: `=~` and `in~` are case-insensitive comparisons; `has` matches whole terms efficiently. `Caller` and `CallerIpAddress` are the columns you will map to entities — a detection without entity-mappable columns produces alerts that investigation tools can't pivot on.

## Exercise 3 — Create the scheduled analytics rule (~10 min)

1. Sentinel → **Configuration → Analytics → Create → Scheduled query rule**.
2. **General**: Name `LAB - Suspicious resource group creation`, Severity **Medium**, MITRE tactic **Impact** (or Persistence — discuss: which fits resource creation?). Status Enabled.
3. **Set rule logic**: paste the Exercise 2 query (with the `soc-lab-alert` filter back in).
4. **Entity mapping** (the step most people skip — don't):
   - Entity **Account** → identifier `FullName` → column `Caller`
   - Entity **IP** → identifier `Address` → column `CallerIpAddress`
5. **Query scheduling**: run every **5 minutes**, lookup data from the last **5 minutes**. (5 min is the minimum for scheduled rules — note it; NRT exists for faster.)
6. **Alert threshold**: greater than 0. **Event grouping**: *Group all events into a single alert*.
7. **Incident settings**: Create incidents = Enabled. Alert grouping = Enabled, group alerts into a single incident if all entities match, within 5 hours.
8. Review + Create.

**Checkpoint**: the rule appears under Analytics → Active rules.

## Exercise 4 — Trigger the detection (~15 min, includes waiting)

1. Azure portal → **Resource groups → Create**: name **`soc-lab-alert-rg`** → Create.
2. Wait. Two delays stack: activity-log ingestion (~5–15 min) + the rule's 5-minute cycle.
3. Watch **Sentinel → Threat management → Incidents** (or the unified **Defender portal → Investigation & response → Incidents** — same incidents, unified queue).

**Checkpoint**: an incident named `LAB - Suspicious resource group creation` appears, severity Medium.

Open it and verify:
- The **Entities** tab shows your account (Account entity) and your IP (IP entity) — this is your Exercise 3 step 4 paying off.
- The **Alerts** tab shows one alert; drill in to see the events.
- Click your IP entity → the entity page opens with related activity and context (the same pivot pattern as in the Defender XDR investigation guides in this repo).

**If nothing fires after 25 minutes**: Analytics → your rule → check *Rule insights / runs*; re-run the Exercise 2 query manually over the last hour — if the query returns the event but no incident exists, the usual culprit is the lookback window vs. ingestion delay (extend lookback to 15 min).

## Exercise 5 — Automation rule: triage on creation (~10 min)

1. Sentinel → **Configuration → Automation → Create → Automation rule**.
2. Name: `LAB - Auto-triage suspicious RG incidents`.
3. **Trigger**: When incident is created.
4. **Conditions**: Analytic rule name → Contains → `LAB - Suspicious resource group creation`.
5. **Actions**, in order:
   - **Change status** → Active
   - **Add tags** → `lab`, `resource-hijack`
   - **Assign owner** → yourself
6. Create. Then trigger again: delete `soc-lab-alert-rg`, wait 2 minutes, recreate it.

**Checkpoint**: the new incident arrives pre-assigned, tagged, and Active — no human touched it. This is the detection-vs-reaction split: the analytics rule *detected*; the automation rule *reacted*. A playbook (Logic App) would be the next escalation step for actions beyond Sentinel (Teams message, enrichment) — wire one via the *Run playbook* action if you want the extension exercise.

## Exercise 6 (optional) — NRT variant (~10 min)

Recreate the rule as **NRT query rule** (Analytics → Create → NRT query rule), same KQL. Notice while configuring:
- No scheduling section — NRT runs every minute by design.
- Try adding a `join` to a second table — observe the single-table constraint in action.

Trigger again and compare detection latency against the scheduled rule.

## Exercise 7 — Cleanup

1. Delete resource groups `soc-lab-alert-rg` and `test-noise-rg`.
2. Disable or delete both analytics rules and the automation rule.
3. To stop ingestion: remove the Azure Activity diagnostic setting (Monitor → Activity log → Export activity logs) and the policy assignment.
4. To remove everything: delete `rg-sentinel-lab` (removes workspace + Sentinel).

---

## What you exercised, mapped to the SC-200 outline

| Lab step | Exam skill |
|---|---|
| Exercise 1 | Configure and manage data connectors / ingestion |
| Exercises 2–3 | Configure scheduled analytics rules, entity mapping, alert & incident grouping, MITRE mapping |
| Exercise 4 | Investigate incidents, entities, and entity pages |
| Exercise 5 | Automation rules: triggers, conditions, ordered actions |
| Exercise 6 | NRT rules and their constraints |

## See also

- [Sentinel Detection Flow — concept guide and diagram](sentinel-detection-flow.md)
- [Scheduled analytics rules in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-custom)
- [Automate incident handling with automation rules](https://learn.microsoft.com/en-us/azure/sentinel/automate-incident-handling-with-automation-rules)
- [Connect Azure Activity logs](https://learn.microsoft.com/en-us/azure/sentinel/data-connectors/azure-activity)
