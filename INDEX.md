# Infrastructure-Level DNS/DHCP Stability (Technitium-first)

**Author:** John Majerus · GitHub: jmajerus@github.com

**Goal:** eliminate NXDOMAIN flaps and brittle naming for infrastructure hosts (LXCs, NAS, tuners, DVR/EPG boxes) while keeping DDNS hygiene for roaming clients.

### The Core Ideas
- **Sticky DDNS for infrastructure**: never delete A/AAAA on lease *expiry*; prefer delete on *release*; allow a grace window before scavenging.
- **Reservation ≠ dynamic client**: treat reservations as first-class *infrastructure* with dedicated policy (pin hostname, longer TTLs, conflict guard).
- **Clear provenance & audit**: tag DNS records (`static(admin)`, `ddns(dynamic)`, `ddns(reservation)`), and log RENEW/RELEASE/EXPIRY events.
- **Usable UI**: give Reservations a top-level tab with search, filters, drawer edit, and bulk actions.

### Why This Matters
Home/SMB services (database, MQTT, IOT services, HomeAssistant, printers, cameras, tuners, etc.) assume names like `service_name.localdomain` *always resolve*. Enterprise-leaning DDNS defaults that delete on *expiry* cause dangling CNAMEs and guide/API failures—even while devices are up.

### What’s in this repo
- **Overview:** see [`proposals-overview.md`](./proposals-overview.md) for a one‑page summary and bundles.
- **Core specs:** [`infra-ddns.md`](./infra-ddns.md), [`reservations-ux.md`](./reservations-ux.md)
- **Strategy & companion:** [`future-direction.md`](./future-direction.md), [`human-assisted-consistency.md`](./human-assisted-consistency.md)
- **Vendor-facing proposals:** [`proposals/feature-infrastructure-reservations-sticky-ddns.md`](./proposals/feature-infrastructure-reservations-sticky-ddns.md), plus technical and summary briefs in `proposals/`
- **Issue forms & PR template:** live in [`.github/`](./.github/) to collect future ideas consistently

#### Where things live
- Root: strategy, specs, and reference docs you read first
- `proposals/`: vendor‑facing requests you can file upstream
- `GLOSSARY.md`: precise definitions so we don’t conflate DHCP leases/reservations with DNS record types

### Status
This is a living proposal. PRs welcome: concrete UI text, API sketches, test plans.

```
Quick start for contributors
- Open a Feature request using the Issue form
- Fork, branch, and submit PRs against `*.md` (and `proposals/*` for vendor-facing items)
- Keep examples generic; redact any private IPs/names
```
```
Project conventions
- Zone name examples use `localdomain`
- TTLs in seconds (e.g., infra = 86400s)
- MAC addresses uppercase colon-delimited
```
```
Non-goals
- Replacing Technitium’s code here; this is design/policy text
- Vendor-specific flame wars; we start with Technitium because many homelabs use it
```
```
License
- MIT (see LICENSE)
```

## Suggested Repo Names
If you want to rename the repo later, here are a few descriptive options:
- `infra-dns-stability` (current)
- `dns-ddns-infrastructure-patterns`
- `technitium-infra-ddns`
- `homelab-dns-dhcp-infra`
- `sticky-ddns-and-reservations`

## Changelog
See **CHANGELOG.md** for dated updates (initialized 2025-10-21).

## TL;DR for filing upstream

Copy/paste this into a vendor issue (see full context in `proposals-overview.md`).

## Document types

- Core specs (how it should work): `infra-ddns.md`, `reservations-ux.md`
- Strategy and rationale (why): `future-direction.md`
- Companion (intelligent assistance): `human-assisted-consistency.md`
- Overview (start here): `proposals-overview.md`
- Vendor-facing proposals (ready to file): `proposals/*`

## Terminology (quick reference)

See `GLOSSARY.md` for full definitions. In brief:
- Dynamic lease vs fixed reservation: different intent, different cleanup policy.
- DHCP‑generated DNS records (DDNS) vs admin‑created static records: never conflate; provenance matters for cleanup and audit.

```
Problem: A/AAAA records get deleted on DHCP lease EXPIRY, causing NXDOMAIN for infrastructure hosts (reservations) during normal churn.

Ask:
- Default to Delete‑on‑RELEASE (on) and Delete‑on‑EXPIRY (off)
- Per‑scope: “Do not delete DNS A/AAAA on lease expiry”
- Grace window (e.g., 24–48h) before any EXPIRY scavenging
- Optional: PTR cleanup as a separate toggle

Rationale: Stability‑first retention. The Oct 2025 AWS DNS outage shows aggressive, context‑blind DNS automation can cascade into outages. Bias toward continuity when uncertain.

Outcome: Infra names remain stable; dynamic hygiene preserved; admins can opt‑in to EXPIRY cleanup with guardrails.
```
