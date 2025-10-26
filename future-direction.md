# Executive Summary

This proposal requests enhancements to Technitium DNS Server that move beyond simple automation toggles toward a smarter, intention-aware integration between DNS and DHCP. At present, automatic removal of A/AAAA records on lease expiry can prematurely delete valid infrastructure mappings—particularly in test or mixed environments where administrators shorten lease times or manually define host records. The October 2025 AWS DNS outage, caused by an automation fault that propagated invalid records and disrupted dependent services worldwide, underscores the need for more conservative, context-sensitive DNS handling. By introducing safeguards such as a “Do not delete on lease expiry” option, lease-grace reconciliation, and detection of manual intent, Technitium can evolve into a self-hardening, administrator-aligned system that prioritizes stability, transparency, and resilience over rigid timing rules.

---

# Future Direction: Toward Intention-Aware, Self-Hardening DNS/DHCP Integration

> See also: [Proposals Overview and Implementation Options](./proposals-overview.md) for concrete bundles that can ship incrementally or together.

The immediate request (e.g. preventing automatic removal of A/AAAA records on lease expiry) should be understood as the first step in a broader evolution. The long-term goal is to make Technitium’s DNS/DHCP integration smarter, safer, and more aligned with actual administrator intent — not just raw lease timing.

## 1. Respect for Administrator Intent

The system should distinguish between:
- transient, automatically generated records; and  
- deliberate, human-created or human-edited infrastructure entries.

Once an administrator has taken the time to create or modify a record, that record should be presumed intentional and therefore protected. It should not be subject to immediate purge just because the related DHCP lease was short (for example, during testing, lab exercises, or intentional rapid cycling of IPs).

**Rationale:** No one goes through the trouble of explicitly defining infrastructure records (for things like hypervisors, storage backends, automation controllers, NVRs, etc.) only to want them silently wiped out moments later. Treating manual touch as a signal of intent dramatically reduces accidental breakage.

## 2. Conservative Retention as a Stability Feature

Instead of aggressively deleting DNS data as soon as a lease expires, the system should bias toward continuity. When there is ambiguity (“Is this device truly gone, or was lease time just dialed down temporarily?”), the safer behavior is to retain name-to-address mappings.

This is not laziness — it’s resilience. The recent October 2025 AWS outage, which stemmed from an automation fault in DNS record management, demonstrated how overly aggressive or context-blind DNS updates can ripple outward and disrupt dependent systems at massive scale. The same class of failure exists in miniature in smaller networks: once DNS loses authoritative knowledge about critical devices, every dependent service begins to fall over.

A stability-first retention model makes DNS the anchor of the network rather than a point of fragility.

## 3. Graceful Deletion via Reconciliation Windows

Instead of immediate hard deletion, expired-lease records could enter a reconciliation state:
- The A/AAAA record remains published, but is internally marked as “awaiting confirmation.”
- If the same hostname/MAC/ID reappears (for example, after a reboot, or after a lease reissue at a new IP), Technitium can reconcile instead of forcing the admin to recreate records manually.
- Only after a grace interval (policy-driven) would the system propose final cleanup.

In other words: use “soft delete with auditability,” not blind removal. This dramatically lowers the blast radius of routine DHCP churn.

## 4. Context Profiles / Role Awareness

The DNS/DHCP engine can become profile-aware. Examples:
- An address that has been referenced by other services (for example, known as the NFS server for backups, or referenced in another static mapping) is more likely to be infrastructure and should get longer retention and higher protection.
- An address that was handed to a disposable IoT test device with a 10-minute lease can be treated more aggressively.

Over time, Technitium could surface this as simple selectable behavior profiles (“conservative,” “balanced,” “aggressive”) without forcing the user to micromanage dozens of individual timers and flags.

## 5. Human-Visible Intent, Machine-Enforced Safety

The system should make its decisions visible. Instead of silently deleting or silently overriding, it should surface:
- “This record is being preserved because it appears to be infrastructure.”
- “This record is scheduled for cleanup in 2 hours unless you pin it.”
- “This hostname was manually created and will not be auto-removed by DHCP lease expiry.”

That does two things at once:
1. It reassures the administrator that nothing critical is about to vanish behind their back.
2. It makes the internal logic teachable and predictable, instead of feeling like opaque magic.

## 6. Alignment with Modern Reliability Principles

This direction aligns with lessons from the AWS DNS outage at global scale: DNS automation can no longer be naïvely trusted to be correct simply because it is automated. Automation must become context-sensitive.

Our proposal moves Technitium toward that standard:
- infer intent where intent is obvious,  
- preserve infrastructure records,  
- prefer graceful degradation over catastrophic removal, and  
- provide auditability of any cleanup.

This is not just a quality-of-life improvement. It’s infrastructure hardening.
