# Proposals Overview and Implementation Options

A single-page summary of everything in this repo, with concrete bundles you can ship sooner or combine.

> Bias DNS toward continuity, not aggressive deletion. When in doubt, keep name-to-address mappings. As noted in the repo’s future direction:
>
> “This is not laziness — it’s resilience. The recent October 2025 AWS outage, which stemmed from an automation fault in DNS record management, demonstrated how overly aggressive or context-blind DNS updates can ripple outward and disrupt dependent systems at massive scale. The same class of failure exists in miniature in smaller networks: once DNS loses authoritative knowledge about critical devices, every dependent service begins to fall over.
>
> A stability-first retention model makes DNS the anchor of the network rather than a point of fragility.”

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
  - Reservations tab with search, filters, badges, bulk actions, drawer edit
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

Source: [Infrastructure-Level DDNS Behavior](./infra-ddns.md), [Reservations UX](./reservations-ux.md)

### Bundle C — Provenance and auditability
- UI tags for record origin: `static(admin)`, `ddns(dynamic)`, `ddns(reservation)`
- Audit log entries show cause (RENEW/RELEASE/EXPIRY), host, MAC, IP, scope, TTL
- Action: Promote dynamic A → static

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
