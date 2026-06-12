# Blocking Custom IPs & URLs with Defender for Endpoint

How to block specific IP addresses and URLs on devices using Microsoft Defender for Endpoint custom network indicators — including the advanced feature that must be enabled first, the prerequisites, and how enforcement actually happens.

![Flow: Microsoft 365 E5 includes Defender for Endpoint P2, which requires network protection in block mode and Defender AV with cloud protection; the Custom network indicators advanced feature unlocks IP/URL indicators with a Block action, enforced via SmartScreen and network protection](../assets/mde-blocking-custom-ips-urls.svg)

## How to?

To block specific IP addresses and URLs with Defender for Endpoint, enable the advanced feature **Custom network indicators** (Microsoft Defender portal → **Settings → Endpoints → Advanced features**), then create indicators with a **Block** action under **Settings → Endpoints → Indicators**. Until the advanced feature is on, IP and URL indicators will not enforce.

## Licensing

Microsoft 365 E5 includes **Defender for Endpoint Plan 2**, which provides full custom indicator support with response actions (block, warn, audit) and device-group scoping.

## Prerequisites

Custom network indicator enforcement depends on the endpoint protection stack:

- **Network protection in block mode** — not audit mode. In audit mode, connections are logged but never blocked. If indicators "aren't working," check this first.
- **Microsoft Defender Antivirus** active (or in EDR block mode scenarios, appropriately configured) with **cloud-delivered protection** enabled.

## Configuration steps

1. Microsoft Defender portal → **Settings → Endpoints → Advanced features** → toggle **Custom network indicators** to **On**.
2. **Settings → Endpoints → Indicators** → choose the **IP addresses** or **URLs/Domains** tab → **Add item**.
3. Specify the indicator value, the **action** (Block and remediate / Warn / Audit), an expiration if appropriate, and the **device groups** it applies to.

## How enforcement works

- **Microsoft Edge** enforces URL/domain indicators through **SmartScreen**.
- **All other browsers and processes** are enforced by **network protection** — which is exactly why network protection in block mode is a hard prerequisite. The telltale symptom of a missing prerequisite: blocks work in Edge but not in Chrome or other apps.

## Limitations worth knowing

- IP indicators support single IPv4/IPv6 addresses — not CIDR ranges. For range-based or large-scale threat-intel-driven blocking, push indicators programmatically via the Defender for Endpoint **Indicators API** or a threat intelligence integration.
- There is a per-tenant indicator limit, so curate rather than dump entire feeds into the portal.
- Web content filtering (category-based blocking) is a separate, complementary feature — indicators are for *specific* IOCs.

## References

- [Create indicators for IPs and URLs/domains](https://learn.microsoft.com/en-us/defender-endpoint/indicator-ip-domain)
- [Configure advanced features in Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/advanced-features)
- [Turn on network protection](https://learn.microsoft.com/en-us/defender-endpoint/enable-network-protection)
- [SC-200 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-200)
