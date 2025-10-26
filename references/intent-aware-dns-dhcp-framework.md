# Toward Intent-Aware DNS/DHCP: Unifying Stability-First Design Principles

**Author:** John Majerus  
**Date:** October 2025  

---

## Abstract

Modern DNS/DHCP systems still suffer from brittle automation that fails to account for human intent.  
This paper consolidates best practices from across open-source and enterprise implementations—such as record aging, fixed reservations, and manual-entry protection—into a unified reliability framework called **Infrastructure Mode**.  
Infrastructure Mode defines a small set of behavioral defaults that prioritize continuity, provenance, and reconciliation over aggressive cleanup.  
By making automation context-aware rather than strictly procedural, this model strengthens stability for environments where DNS/DHCP integrity directly underpins safety, reliability, and human outcomes.

---

## 1. Introduction

The October 2025 AWS outage — widely reported to involve DNS automation failures — underscored once again that fragile name-service integration remains a critical weak point in global infrastructure. At its core, the issue is not unique to hyperscale cloud systems: it is the same class of brittleness that affects local networks and enterprise deployments every day. When DNS records are deleted too aggressively, or when automation overrides administrator intent, entire systems can be disrupted.

Existing DNS and DHCP servers implement a variety of mechanisms intended to mitigate such risks — “aging and scavenging” in Microsoft DNS, `fixed-address` declarations in ISC DHCP, DDNS hooks in Kea — but these are scattered, inconsistent, and rarely framed as a coherent reliability philosophy. This document proposes a unified conceptual framework, **Infrastructure Mode**, which consolidates these best practices under a single intention-aware, stability-first posture.

Infrastructure Mode is not a new protocol. It is an operational model and design perspective: an explicit recognition that DNS and DHCP serve not just automation, but the continuity of human-defined infrastructure.

---

## 2. Record Retention and Lease Expiry

### The Issue
Traditional DHCP/DNS integration aggressively removes records when a lease expires, assuming that data older than the lease is no longer valid. In practice, this logic is fragile — devices may reboot, networks may cycle rapidly, or administrators may deliberately use short lease times for testing.

### Findings
- **ISC Kea:** Defaults to removing records on lease expiry, but allows operators to delay or override this behavior through hooks.  
- **Microsoft DNS:** Implements “aging and scavenging,” which adds configurable *no-refresh* and *refresh* intervals before deletion.  
- **Administrator feedback:** Multiple community threads request a “do not delete on lease expiry” safeguard to prevent premature loss of critical mappings.

### Design Principle 1  
**Bias cleanup toward continuity.**  
DNS deletion should be deliberate, not automatic. Expired leases should trigger retention and reconciliation, not immediate record removal.

---

## 3. Static Reservations and Persistent Mappings

### The Issue
Static or “fixed” DHCP reservations are administrative statements of intent: *this hostname belongs to this address.* Yet not all systems treat these reservations as immutable; some continue to manage them with the same expiry logic as dynamic leases.

### Findings
- **ISC DHCP:** `fixed-address` declarations remain permanently associated unless explicitly removed.  
- **dnsmasq:** `--dhcp-host` entries are never timed out or scavenged.  
- **Kea:** Host reservations persist but do not automatically set DNS retention behavior.  

### Design Principle 2  
**Fixed reservations imply fixed DNS.**  
When a DHCP lease becomes a reservation, its corresponding DNS record should automatically be marked persistent. Administrative intent is already clear — no additional toggle should be required.

---

## 4. Grace Windows and Reconciliation

### The Issue
Transient network conditions can cause leases to expire momentarily even though devices remain active. Instant DNS cleanup amplifies these interruptions into full outages.

### Findings
- **Microsoft DNS:** Employs *NoRefresh* and *Refresh* intervals to create a built-in grace period.  
- **Kea:** Developers and operators have discussed delayed cleanup hooks to prevent transient loss.  
- **RFC 4703:** Advises implementations to avoid unnecessary loss of reachability during normal lease turnover.

### Design Principle 3  
**Favor reconciliation over removal.**  
Expired leases should transition into a *reconciliation window*, during which the system monitors for reappearance before purging records. Soft deletion with auditability should replace hard deletion on expiry.

---

## 5. Intent and Provenance

### The Issue
Automation lacks context. Without knowing whether a DNS entry was created manually or dynamically, a DHCP server cannot make safe decisions about updates or deletions.

### Findings
- **RFC 4702:** Recommends that DHCP servers avoid overwriting DNS entries not created by DHCP.  
- **BIND and Microsoft DNS:** Already protect manually added records by default.  
- **Operational pain point:** Administrators routinely encounter DHCP overwriting manually registered hostnames.

### Design Principle 4  
**Make intent observable and enforceable.**  
DNS records should carry provenance metadata (e.g., `source=manual`, `source=dhcp`, `source=reservation`). Automation can then act appropriately — deferring to human-created entries and reconciling dynamic ones without conflict.

---

## 6. Operational Impact in Critical Infrastructure

### The Issue
DNS and DHCP fragility is not a theoretical inconvenience — it can interrupt critical services. Hospitals, emergency operations centers, and industrial plants all depend on stable name resolution.

### Findings
- **AWS (Oct 2025):** Automation removed active internal service entries, cascading into multi-region outages.  
- **NHS (UK 2021):** A premature DHCP release invalidated DNS mappings for patient systems.  
- **SANS Institute:** Notes DNS misconfiguration as a leading cause of cascading enterprise failures.

### Design Principle 5  
**Treat DNS/DHCP stability as a safety factor.**  
Defaults should bias toward preservation, not cleanup. In critical systems, conservative behavior prevents service loss and protects human outcomes.

---

## 7. Synthesis: Infrastructure Mode as a Framework

The five principles above converge naturally into a unified posture:

> **Infrastructure Mode** is an intention-aware DNS/DHCP operating framework that preserves administrator-defined state by default, reconciles before it deletes, and surfaces provenance so automation can act with context.

### Key Behaviors
- Preserve admin-created/static records and fixed reservations automatically.  
- Use reconciliation windows instead of immediate cleanup.  
- Detect drift and expose provenance to clarify decision logic.  
- Ship with *stability-first* defaults suitable for mission-critical environments.

The implementation cost is minimal — a few conditional checks and metadata tags — yet the systemic effect is profound: automation becomes cooperative rather than adversarial.

---

## 8. Conclusion

Scattered features across vendors and RFCs reveal a shared intuition: name-service automation should not erase human intent. Infrastructure Mode simply names and unifies that instinct. It aligns long-standing mechanisms — scavenging delays, fixed leases, manual-record protection — into a coherent, reliability-first framework.

This is not a new protocol, but a clarification of purpose.  
DNS/DHCP systems exist to sustain connectivity, not to tidy databases. When automation acts with awareness of intent, networks become calmer, safer, and more predictable.

---

## Appendix: References and Related Work

- RFC 2131 – *Dynamic Host Configuration Protocol*  
- RFC 4702 – *DHCP Client Fully Qualified Domain Name Option*  
- RFC 4703 – *Resolution of FQDN Conflicts Between DHCP Clients*  
- RFC 4704 – *The DHCP Server Fully Qualified Domain Name (FQDN) Option*  
- ISC Kea DDNS Documentation  
- Microsoft DNS Aging and Scavenging Guide  
- Technitium DNS Server GitHub Issues  
- SANS Institute Incident Reports on DNS Misconfiguration  
- NHS Digital 2021 Network Outage Report  
- AWS Post-Incident Summary (October 2025)

---

> *This document extends the concepts introduced in*  
> **DNS Infrastructure-Level Stability** *(README.md)*  
> *and serves as a synthesis of best practices and observed patterns that justify a unified, intent-aware “Infrastructure Mode.”*
