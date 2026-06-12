# Attack Surface Reduction (ASR) Rules in Defender for Endpoint

How to block executable content from email messages and unsigned processes running from USB drives using Microsoft Defender for Endpoint — the capability to use, the prerequisites, deployment modes, and how to monitor rule events.

![Flow: Microsoft 365 E5 includes Defender for Endpoint, whose ASR rules capability contains built-in rules for blocking executable email content and untrusted/unsigned USB processes, requiring Defender Antivirus as primary AV, with Block/Audit/Warn modes deployed via Intune, GPO, or PowerShell](../assets/mde-asr-rules.svg)

## The scenario

> You have a Microsoft 365 E5 subscription. You plan to deploy Microsoft Defender for Endpoint to: block executable content from email messages, and block unsigned processes that run from USB drives.

**Answer: attack surface reduction (ASR) rules.**

The giveaway: both requirements are near-verbatim names of built-in ASR rules:

- **Block executable content from email client and webmail**
- **Block untrusted and unsigned processes that run from USB**

When requirements read like rule names, the question is testing whether you recognize the ASR rule catalog.

## What ASR rules are

ASR rules are a set of roughly 16 built-in behavioral controls in Defender for Endpoint that constrain risky behaviors commonly abused by malware — Office apps spawning child processes, scripts launching downloaded executables, credential theft from LSASS, and so on. Each rule is identified by a GUID and is configured independently.

Other rule names worth recognizing on sight:

- Block Office applications from creating child processes
- Block credential stealing from the Windows local security authority subsystem (lsass.exe)
- Block JavaScript or VBScript from launching downloaded executable content
- Block process creations originating from PSExec and WMI commands
- Use advanced protection against ransomware

## Prerequisites

ASR rules require **Microsoft Defender Antivirus as the primary (active) antivirus**. If a third-party AV is primary and Defender AV runs in passive mode, ASR rules do not apply — the classic "we configured the rules but nothing is blocked" twist.

## Modes and rollout

Each rule runs in one of three modes:

| Mode | Behavior |
|---|---|
| Audit | Events are logged, nothing is blocked — use for impact assessment |
| Block | The behavior is prevented |
| Warn | The user is warned and can choose to bypass (not supported by every rule) |

Recommended rollout: deploy in **Audit** mode, review the generated events, add exclusions for legitimate business processes, then switch to **Block**.

## Deployment

ASR rules are deployed via **Intune** (Endpoint security → Attack surface reduction policies), **Group Policy**, or **PowerShell**:

```powershell
Set-MpPreference -AttackSurfaceReductionRules_Ids <rule GUID> -AttackSurfaceReductionRules_Actions Enabled
```

## Monitoring with Advanced Hunting

ASR events land in the **`DeviceEvents`** table (not `DeviceProcessEvents`) — `DeviceEvents` is the catch-all table for events without a dedicated table, including ASR, AMSI, and USB activity:

```kql
DeviceEvents
| where ActionType startswith "Asr"
| summarize count() by ActionType, FileName, DeviceName
| order by count_ desc
```

The Defender portal also provides ASR reports under **Reports → Attack surface reduction rules** for audit-to-block impact analysis.

## Distractors to rule out

- **Network protection** — governs outbound web/IP traffic, not local process behavior.
- **Controlled folder access** — protects designated folders from untrusted apps (anti-ransomware), not email or USB execution.
- **Device control** — blocks or restricts the USB storage device itself; the scenario asks about *processes running from* USB, which is the ASR rule's job.

## References

- [Attack surface reduction rules overview](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction)
- [ASR rules reference (full rule list and GUIDs)](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference)
- [Enable attack surface reduction rules](https://learn.microsoft.com/en-us/defender-endpoint/enable-attack-surface-reduction)
