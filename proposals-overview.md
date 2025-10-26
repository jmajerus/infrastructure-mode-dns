# Proposals Overview and Implementation Options

## TL;DR (copy‑paste for vendor issues)

Problem: A/AAAA records get deleted on DHCP lease EXPIRY, causing NXDOMAIN for infrastructure hosts (reservations) during normal churn. This breaks dependent services.

Ask:
- Default to Delete‑on‑RELEASE (on) and Delete‑on‑EXPIRY (off)
- Add per‑scope: “Do not delete DNS A/AAAA on lease expiry”
- Add a grace window (e.g., 24–48h) before any EXPIRY scavenging
- Optional: PTR cleanup as a separate toggle
 - When creating/toggling a fixed DHCP reservation, auto‑promote any existing A/AAAA to admin static and pin the hostname (preserve PTR; record provenance)
 - Global setting: when admin creates static A/AAAA for an in‑scope IP, Prompt/Auto‑create a fixed reservation; otherwise watch/flag drift

Default safety rails (recommended; vendor sets actual defaults):
- Delete‑on‑RELEASE = ON; Delete‑on‑EXPIRY = OFF; Grace window = ON (e.g., 24–48h)
- Infrastructure mode available on reservations with Pin Hostname, TTL=86400, Conflict Guard = ON
- Auto‑promote to static on reservation save = ON
- DNS‑first policy = Prompt (offer to create reservation); Auto‑watch = ON
- Drift detection = ON (Warn by default; Block optional)

Rationale: Stability‑first retention, in service of making it easy and safe to create and maintain infrastructure components (fixed reservations + sticky DNS). The October 2025 AWS DNS outage showed how aggressive, context‑blind DNS automation can cascade into outages. Bias toward continuity when uncertain.

Outcome: Infrastructure names remain stable; dynamic hygiene is preserved via RELEASE; admins can opt‑in to EXPIRY cleanup with guardrails.

A single-page summary of everything in this repo, with concrete bundles you can ship sooner or combine.

Terminology used (see `GLOSSARY.md`):
- Dynamic lease (no reservation) vs fixed reservation (user‑designated) — policies differ.
- DHCP‑generated DNS records (DDNS) vs admin‑created static records — provenance matters for cleanup.

> Bias DNS toward continuity, not aggressive deletion. When in doubt, keep name-to-address mappings. As noted in the repo’s future direction:
>
> “This is not laziness — it’s resilience. The recent October 2025 AWS outage, which stemmed from an automation fault in DNS record management, demonstrated how overly aggressive or context-blind DNS updates can ripple outward and disrupt dependent systems at massive scale. The same class of failure exists in miniature in smaller networks: once DNS loses authoritative knowledge about critical devices, every dependent service begins to fall over.
>
> A stability-first retention model makes DNS the anchor of the network rather than a point of fragility.”

Preferred path vs alternatives:
- Preferred: Fixed DHCP reservation + auto‑promote/pin DNS (becomes an “infrastructure component”).
- Alternatives: Admin static DNS with true static IPs outside DHCP; IPAM‑managed static mappings; manual pin/promotion.
- Trade‑off: Alternatives work, but skip DHCP automation and provenance; the fixed‑reservation path offers better auditability and fewer footguns.

## Themes at a glance

- Stability-first retention
  - Default to Delete-on-RELEASE; make Delete-on-EXPIRY opt-in with a grace window
  - Optional soft-delete (reconciliation window) instead of immediate removal
- Infrastructure reservations ("Infrastructure" mode)
  - Sticky DNS (never delete on EXPIRY), Pin Hostname, TTL class (e.g., 86400s), Conflict Guard
- Provenance and auditability
  - Record tags (static/admin, ddns/dynamic, ddns/reservation); audit events (RENEW/RELEASE/EXPIRY)
  - Promote dynamic A → static
- Operator UX
  - Dedicated Fixed Reservations tab (separate from Active Leases) with search, filters, badges, bulk actions, drawer edit; promote lease → reservation flow
- Context profiles
  - Conservative / Balanced / Aggressive behaviors without micromanaging timers

## Implementation bundles (ship together or incrementally)

These group related changes that deliver clear user value with manageable surface area.

### Bundle A — Quick wins (minimal code surface)
- Global defaults: Delete-on-RELEASE = enabled; Delete-on-EXPIRY = disabled
- Per-scope flag: “Do not delete DNS A/AAAA on lease expiry”
- Grace window before any EXPIRY scavenging (e.g., 24–48h)
- Optional: PTR cleanup as a separate toggle

Acceptance examples:
- Infra names never disappear solely due to EXPIRY
- Dynamic hygiene preserved via RELEASE; admins can opt into EXPIRY cleanup with grace

Source: [Infrastructure-Level DDNS Behavior](./infra-ddns.md), [Feature Request](./proposals/feature-request.md)

### Bundle B — Infrastructure reservations (sticky + safe)
- Reservation-level Infrastructure mode:
  - Sticky DNS (no delete on EXPIRY)
  - Pin Hostname (ignore client-reported names for infra)
  - TTL Class: Infrastructure (e.g., 86400s)
  - Conflict Guard (reject DDNS from mismatched MAC unless approved)

Acceptance examples:
- Marking a reservation as Infrastructure guarantees stable A/AAAA retention
- Name remains consistent regardless of client-reported hostname churn
- Operators manage reservations in a dedicated tab; dynamic leases are visible separately
- Creating or toggling a fixed reservation auto‑promotes any existing A/AAAA to admin static and pins hostname at that moment (with provenance recorded)

Source: [Infrastructure-Level DDNS Behavior](./infra-ddns.md), [Reservations UX](./reservations-ux.md)

### Bundle C — Provenance and auditability
- UI tags for record origin: `static(admin)`, `ddns(dynamic)`, `ddds(reservation)`
- Audit log entries show cause (RENEW/RELEASE/EXPIRY), host, MAC, IP, scope, TTL
- Action: Promote dynamic A → static

Also add DNS‑first guardrails:
- When an admin creates a static A/AAAA, offer to create a fixed DHCP reservation for that IP if it’s in a DHCP scope; otherwise watch/flag drift.
- Warn on attempts to reassign a watched IP or overwrite a pinned hostname.

Acceptance examples:
- Admins can see at a glance why a record exists and what changed it
- One click to freeze a dynamic mapping into static when needed

Source: [Infrastructure-Level DDNS Behavior](./infra-ddns.md), [Reservations UX](./reservations-ux.md)

### Bundle D — Soft delete with reconciliation (safer expiry)
- On EXPIRY, mark records as “awaiting confirmation” and keep them published during a grace window
- Reconcile automatically if the same hostname/MAC returns
- After grace, propose cleanup (with audit)

Acceptance examples:
- Routine DHCP churn no longer causes accidental outages
- Cleanup remains possible, but visible and reversible during the window

Source: [Future Direction](./future-direction.md)

### Bundle E — Context profiles (policy presets)
- Ship selectable profiles: Conservative, Balanced, Aggressive
- Profiles adjust retention, grace windows, and overwrite/conflict posture without per-flag micromanagement

Acceptance examples:
- Admins pick a profile matching environment needs; can still tune advanced options

Source: [Future Direction](./future-direction.md)

## "Can ship together" combos

- A + B: Highest user value quickly — stable infra plus sensible defaults
- A + C: Stability plus observability/debuggability
- B + C + D: Infra stickiness, clarity, and safer expiry behavior (ideal for mixed environments)
- E overlays the above with approachable presets

## Minimal viable path (if picking just one)

Start with Bundle A. It flips the dangerous default and adds a grace window; it’s small, testable, and solves the most painful outage class.

## Cross-links to details

- Core behavior: [Infrastructure-Level DDNS Behavior](./infra-ddns.md)
- UX flows: [Reservations UX Proposal](./reservations-ux.md)
- Strategy/principles: [Future Direction](./future-direction.md)
- Vendor-facing write-ups:
  - [Feature Request: Preserve DNS on Lease Expiry](./proposals/feature-request.md)
  - [Technical Analysis: DHCP/DNS Ownership Conflict](./proposals/feature-dns-retention-technical.md)
  - [Summary: Delete-on-RELEASE vs Delete-on-EXPIRY + Infrastructure Mode](./proposals/feature-dns-retention-summary.md)

## Notes for reviewers

- Examples use `localdomain`; TTLs shown in seconds (infra default: 86400)
- Keep infra on sticky DNS; use short TTLs for dynamic clients
- Avoid mDNS `.local` conflicts; publish DHCP Option 6 (DNS) and 15 (domain) consistently
- Companion read: [Collaborative DNS/DHCP Intelligence — Respecting Intent While Preventing Human Error](./human-assisted-consistency.md)
