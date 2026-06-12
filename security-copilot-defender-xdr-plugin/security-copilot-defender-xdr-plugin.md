# Security Copilot: The Defender XDR Plugin and the Embedded Experience

The Microsoft Defender XDR plugin must be enabled in Security Copilot for Copilot to invoke Defender XDR product-specific capabilities from the embedded experience in the Microsoft Defender portal. Without the plugin, Copilot features such as incident summaries and guided responses are unavailable within Microsoft Defender XDR.

![Flow: the Defender XDR plugin enabled in Security Copilot (with provisioned SCU capacity) invokes XDR capabilities that power the embedded Copilot features in the Defender portal — incident summaries, guided responses, script analysis, NL-to-KQL, and incident reports](../assets/security-copilot-defender-xdr-plugin.svg)

## The dependency chain

The embedded Copilot experience in the Defender portal is **not self-contained** — it is a front-end to Security Copilot, and Security Copilot only reaches Defender XDR through the **Microsoft Defender XDR plugin**:

1. **Capacity**: Security Copilot requires provisioned capacity (**Security Compute Units, SCUs**). No SCUs, no Copilot anywhere.
2. **Plugin**: the **Microsoft Defender XDR plugin** must be enabled in Security Copilot (**Sources → Manage plugins**). This is what allows Copilot to invoke XDR product-specific capabilities.
3. **Permissions**: the user needs appropriate roles in both Security Copilot and Defender XDR.

Break any link and the embedded features stop working — but the plugin is the answer when the symptom is "Copilot works in the standalone portal, but incident summaries / guided responses are missing in the Defender portal."

## What the plugin enables (embedded features)

- **Incident summaries** — generated summary cards on incident pages
- **Guided responses** — recommended, ordered response actions on incidents
- **Script and file analysis** — explain suspicious PowerShell/scripts and files
- **Natural language to KQL** — generate Advanced Hunting queries from plain language
- **Incident reports** — generated post-incident reporting

## Standalone vs. embedded

| | Standalone | Embedded |
|---|---|---|
| Surface | securitycopilot.microsoft.com | Inside the Microsoft Defender portal |
| Interaction | Prompts, promptbooks, multi-plugin sessions | Contextual: cards and panes on incidents, hunting, file pages |
| Plugin dependency | Many plugins across products | Same Defender XDR plugin underneath |

Same Copilot, same plugin dependency, different surface. The current SC-200 outline frames this as *investigating incidents by using agentic AI, including embedded Copilot* — so expect scenarios about the embedded experience and what makes it function.

## Distractor to watch

"Enable the plugin in the Defender portal settings" — wrong direction. The plugin lives in **Security Copilot's** settings; the Defender portal consumes it.

## References

- [Microsoft Copilot in Microsoft Defender](https://learn.microsoft.com/en-us/defender-xdr/security-copilot-in-microsoft-365-defender)
- [Manage plugins in Microsoft Security Copilot](https://learn.microsoft.com/en-us/copilot/security/manage-plugins)
- [Get started with Microsoft Security Copilot](https://learn.microsoft.com/en-us/copilot/security/get-started-security-copilot)
- [SC-200 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-200)
