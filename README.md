# Infrastructure-Level DNS/DHCP Stability (Technitium-first)

**Author:** John Majerus · GitHub: jmajerus@github.com

**Goal:** eliminate NXDOMAIN flaps and brittle naming for infrastructure hosts (LXCs, NAS, tuners, DVR/EPG boxes) while keeping DDNS hygiene for roaming clients.

### The Core Ideas
- **Sticky DDNS for infrastructure**: never delete A/AAAA on lease *expiry*; prefer delete on *release*; allow a grace window before scavenging.
- **Reservation ≠ dynamic client**: treat reservations as first-class *infrastructure* with dedicated policy (pin hostname, longer TTLs, conflict guard).
- **Clear provenance & audit**: tag DNS records (`static(admin)`, `ddns(dynamic)`, `ddns(reservation)`), and log RENEW/RELEASE/EXPIRY events.
- **Usable UI**: give Reservations a top-level tab with search, filters, drawer edit, and bulk actions.

### Why This Matters
Home/SMB services (Jellyfin, Dispatcharr, HomeAssistant, printers, cameras, tuners) assume names like `epg.localdomain` *always resolve*. Enterprise-leaning DDNS defaults that delete on *expiry* cause dangling CNAMEs and guide/API failures—even while devices are up.

### What’s in this repo
- **`docs/infra-ddns.md`** — the feature spec for DDNS behavior.
- **`docs/reservations-ux.md`** — the UI/UX proposal for a standalone Reservations tab.
- **`proposals/feature-request.md`** — a copy-paste vendor issue with acceptance criteria.
- **Issue forms & PR template** — to collect future ideas consistently.

### Status
This is a living proposal. PRs welcome: concrete UI text, API sketches, test plans.

```
Quick start for contributors
- Open a Feature request using the Issue form
- Fork, branch, and submit PRs against `docs/*`
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
