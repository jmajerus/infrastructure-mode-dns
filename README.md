# DNS Infrastructure-Level Stability
**Rethinking DNS/DHCP integration for resilience, intent awareness, and automation that serves reliability.**  
**A unified, stability-first framework that makes DNS/DHCP automation respect administrator intent and preserve continuity by default.**  
_Originated from early observations of Technitium DNS Server behavior and underscored by the October 2025 AWS DNS outage._

> [!IMPORTANT]
> **Infrastructure Mode (proposed):** A new, first-class, intention-aware DNS/DHCP operating posture and conceptual framework.  
> Not in current releases. It preserves admin-created/static records and fixed reservations by default, reconciles via grace windows, detects drift, surfaces provenance, and ships with stability-first defaults.  
> **Why this matters:** DNS/DHCP integrity underpins mission- and safety-critical environments (from hospitals and emergency services to manufacturing). Defaults should bias toward stability over aggressive cleanup.

---

## Introduction and Background

In the weeks leading up to the October 2025 AWS outage, I had begun exploring Technitium DNS Server as a core element in building a more resilient and self-maintaining network infrastructure. Almost immediately, I noticed an unexpected brittleness: DNS records that were carefully and deliberately created could disappear simply because of short or expired DHCP leases. From a protocol standpoint, this behavior was technically correct — the DNS data mirrored the state of the DHCP database — yet from an operational perspective it felt unnecessarily fragile.  

The pattern was clear: the software could not distinguish between transient automation and deliberate human intent. When an administrator creates a manual A record or defines a fixed DHCP reservation, that action carries intent. It signals that the record represents something lasting — infrastructure, not ephemera. Losing those records simply because a lease timer expired, or because a short lease was used for testing, undermines the stability of the entire local ecosystem.

Motivated by that insight, I began outlining proposals to make Technitium more context-aware — to treat manually created records and clearly infrastructural hosts with the persistence and respect they deserve. The aim was not to abandon automation but to make it smarter: capable of inferring when stability should take precedence over protocol exactness.  

Then came the AWS outage. Within days of drafting the first notes on these improvements, the world’s largest cloud platform experienced a service-wide disruption traced to automated DNS record management gone wrong. A missing or invalid entry propagated through dependent systems and caused widespread failure — the same class of fragility, but on a global scale. The parallel was unmistakable: if automation without context can topple hyperscale infrastructure, it can certainly disrupt smaller networks.

This coincidence transformed the idea from a local optimization into a broader design principle. DNS and DHCP integration should be **intention-aware**, **infrastructure-centered**, and **self-hardening**. It should recognize when a human has made a deliberate configuration choice and preserve that state through transient automation events. By embedding logic that respects both user intent and operational continuity, Technitium can evolve from a capable DNS/DHCP pair into a genuinely intelligent infrastructure platform — one that learns to protect what administrators value most: reliability, predictability, and trust.

---

## Executive Summary

This proposal introduces Infrastructure Mode — a new, first-class operating posture for Technitium DNS/DHCP — that is not in current releases today. Infrastructure Mode treats administrator-created DNS records and fixed DHCP reservations as durable infrastructure by default, adds conservative reconciliation paths, and surfaces intent and provenance so automation can act safely.

In contrast to today’s lease-led behavior, Infrastructure Mode:
- preserves admin-created/static A/AAAA records and fixed DHCP reservations by default (no auto-delete on lease expiry);
- reconciles via a policy-driven grace window instead of hard-deleting immediately;
- promotes DNS-first awareness: when a static A/AAAA exists, DHCP is prompted/auto-aligned and drift is detected;
- makes decisions visible (why a record is retained, when it will be cleaned, how it was created) with auditability;
- ships with conservative, stability-first defaults (“safety rails”) vendors can tune.

Availability: Infrastructure Mode is a proposed capability. Some behaviors can be approximated with existing per-record flags, but the unified, intention-aware mode described here is not yet fully available. This repository is the design and vendor brief to make it real.

---

## Future Direction: Introducing Infrastructure Mode (Intention-Aware, Self-Hardening)

Infrastructure Mode is a named, explicit operating posture — not just a collection of toggles — that clarifies how DNS and DHCP behave when human intent is evident. It can be implemented incrementally (several parts are small, localized changes), but the value comes from treating the behaviors as a coherent mode that administrators can opt into (and vendors can ship as the recommended default).

The guiding principle is straightforward: when a record is deliberately created or a lease is made fixed, the system infers persistence and protects it without extra steps. This brings the software’s behavior into closer harmony with real-world usage, where infrastructure should not vanish due to transient lease churn.

### 1. Respect for Administrator Intent
The system should distinguish between:
- transient, automatically generated records; and  
- deliberate, human-created or human-edited infrastructure entries.

Once an administrator has taken the time to create or modify a record, that record should be presumed intentional and therefore protected. It should not be subject to immediate purge just because the related DHCP lease was short (for example, during testing, lab exercises, or intentional rapid cycling of IPs).

**Rationale:** No one goes through the trouble of explicitly defining infrastructure records (for things like hypervisors, storage backends, automation controllers, NVRs, etc.) only to want them silently wiped out moments later. Treating manual touch as a signal of intent dramatically reduces accidental breakage.

### ✳️ Fixed Reservations Imply Fixed DNS
When an administrator converts a DHCP lease into a *fixed reservation*, that act itself expresses clear intent:  
> “This hostname and address pairing is important enough to preserve.”

The system should interpret that action as a durable mapping, automatically marking the associated DNS A/AAAA record as *persistent* and exempt from automatic deletion.  

This implicit coupling creates a smoother and safer workflow than requiring a separate “Do not delete on lease expiry” checkbox. The administrator’s natural behavior — converting an ephemeral lease into a fixed reservation — already communicates permanence. The software should honor that signal automatically.  

This approach strengthens infrastructure in two important ways:
1. **Reduced cognitive friction:** one action (making the reservation fixed) communicates everything the system needs to infer user intent.  
2. **Automatic alignment of DNS and DHCP semantics:** fixed DHCP entries and fixed DNS entries stay in sync by design, removing a common source of desynchronization and record loss.

Over time, the “Do not delete on lease expiry” setting could remain as an override for specialized use cases, but the **default assumption** should be that fixed DHCP reservations imply fixed DNS records. In Infrastructure Mode, this is a default invariant; outside the mode, it can remain an opt‑in behavior. That’s the natural, intention-aware path toward a resilient and administrator-friendly system.

### 2. Conservative Retention as a Stability Feature
Instead of aggressively deleting DNS data as soon as a lease expires, the system should bias toward continuity. When there is ambiguity (“Is this device truly gone, or was lease time just dialed down temporarily?”), the safer behavior is to retain name-to-address mappings.

This is not laziness — it’s resilience. The recent October 2025 AWS outage, which stemmed from an automation fault in DNS record management, demonstrated how overly aggressive or context-blind DNS updates can ripple outward and disrupt dependent systems at massive scale. The same class of failure exists in miniature in smaller networks: once DNS loses authoritative knowledge about critical devices, every dependent service begins to fall over.

A stability-first retention model makes DNS the anchor of the network rather than a point of fragility.

### 3. Graceful Deletion via Reconciliation Windows
Instead of immediate hard deletion, expired-lease records could enter a reconciliation state:
- The A/AAAA record remains published, but is internally marked as “awaiting confirmation.”
- If the same hostname/MAC/ID reappears (for example, after a reboot or after a lease reissue at a new IP), Technitium can reconcile instead of forcing the admin to recreate records manually.
- Only after a grace interval (policy-driven) would the system propose final cleanup.

Use **soft delete with auditability**, not blind removal. This dramatically lowers the blast radius of routine DHCP churn.

### 4. Context Profiles / Role Awareness
The DNS/DHCP engine can become profile-aware. Examples:
- An address referenced by other services (e.g., an NFS server or static mapping) is more likely infrastructure and should receive longer retention and higher protection.
- An address handed to a disposable IoT test device with a 10-minute lease can be treated more aggressively.

Over time, Technitium could surface this as simple selectable behavior profiles (“conservative,” “balanced,” “aggressive”) without requiring administrators to micromanage dozens of timers and flags.

### 5. Human-Visible Intent, Machine-Enforced Safety
The system should make its decisions visible. Instead of silently deleting or silently overriding, it should surface:
- “This record is being preserved because it appears to be infrastructure.”
- “This record is scheduled for cleanup in 2 hours unless you pin it.”
- “This hostname was manually created and will not be auto-removed by DHCP lease expiry.”

That does two things at once:  
1. It reassures the administrator that nothing critical is about to vanish behind their back.  
2. It makes the internal logic teachable and predictable, instead of feeling like opaque magic.

### 6. Alignment with Modern Reliability Principles
This direction aligns with lessons from the AWS DNS outage at global scale: DNS automation can no longer be naïvely trusted to be correct simply because it is automated. Automation must become context-sensitive.

Our proposal moves Technitium toward that standard:
- infer intent where intent is obvious,  
- preserve infrastructure records,  
- prefer graceful degradation over catastrophic removal, and  
- provide auditability of any cleanup.

This is not just a quality-of-life improvement — it’s infrastructure hardening.

---

## Conclusion

The issues highlighted here go well beyond Technitium itself. They speak to a broader truth about the fragility of modern network automation: DNS remains the beating heart of digital infrastructure, and yet it is still treated by many systems as a disposable layer governed only by timers and cleanup scripts. The October 2025 AWS outage demonstrated, with global visibility, how even the most advanced infrastructures can be brought to their knees when automation outruns human intent.

By incorporating intention awareness, conservative retention, and infrastructure-centered logic, Technitium has an opportunity to model a new standard — one that other DNS and DHCP implementations could follow. The same guiding principles apply universally: automation should preserve meaning, not erase it; stability should take precedence over procedural tidiness; and human intent should remain the ultimate source of truth in configuration management.

The goal is not to discard automation but to civilize it — to ensure it acts as an intelligent assistant rather than an unthinking enforcer of expiration times. Whether in a home lab, an enterprise LAN, or a hyperscale cloud, the core lesson is the same: a resilient network begins with DNS that understands when to act, when to pause, and when to simply hold steady.

Thinking deeply about how people actually use these systems requires a different mindset than the one that may have historically dominated infrastructure software design. It means moving beyond the engineer’s instinct for procedural neatness and into the realm of human context — where design is guided not just by what the protocol demands, but by what stability, trust, and user intent suggest.

Even a small shift in mindset — one that elevates user experience and administrator intent to the same level of consideration as protocol correctness — can strengthen reliability in meaningful ways. It’s not a wholesale departure from established design principles, but a refinement of them: ensuring that automation respects the human context in which it operates. Approached this way, DNS need not change dramatically to become more dependable; it simply needs to give intent and continuity the attention they have always deserved.


---
## Where to go next

For an organized map of all major files, proposals, and references, see [INDEX.md](./INDEX.md).
