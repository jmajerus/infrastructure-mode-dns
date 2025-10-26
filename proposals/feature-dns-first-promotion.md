# Feature Request: DNS‑First Promotion & Guardrails

## Summary

When an administrator creates a static A/AAAA record, offer to create a corresponding fixed DHCP reservation for that IP if it lies inside a DHCP scope. If auto‑create is enabled, perform it immediately. If the IP is dynamically leased or the client identity is unknown, provide a watch/flag flow and drift alerts.

## Problem

DNS and DHCP intent diverge when operators add static DNS for infrastructure names but DHCP still treats the address as dynamic. This leads to reallocation risk, hostname overwrite attempts, and outages if the lease lapses.

## Proposal

On admin static A/AAAA creation (or selection) in DNS UI:
- If IP ∈ DHCP scope and not reserved: Prompt “Create Fixed Reservation?” [Create] [Dismiss]
- If IP is currently a dynamic lease: Prompt “Dynamic lease; reserve to prevent reallocation.” [Create Reservation]
- If MAC/Client ID unknown: Place IP on a watchlist; prompt to bind when a lease is next observed
- Record provenance: link DNS record to the reservation creation or the watchlist event

Drift detection & alerts:
- Warn/deny on attempts to reassign a watched IP
- Warn/deny on dynamic DDNS updates that attempt to overwrite a pinned hostname

Policy toggles (Global → DNS/DHCP integration):
- On static DNS for in‑scope IP: Prompt | Auto‑create | Ignore (default: Prompt)
- Drift detection: Warn | Block (default: Warn)
- Auto‑watch in‑scope static DNS without reservations (default: On)

## Expected Behavior

- Admin‑created static DNS can be reconciled with DHCP in one click (or automatically per policy)
- Fixed reservations created from DNS inherit Infrastructure defaults (Sticky DNS, Pin Hostname, TTL class, Conflict Guard)
- Alerts surface in both DNS and DHCP views when drift is detected

## Rationale

Bridging DNS‑first workflows reduces operator error, preserves intent, and prevents silent drift between name authority and address authority.

## Related

- Core behavior: `infra-ddns.md`
- UX: `reservations-ux.md`
- Overview bundle: `proposals-overview.md`
