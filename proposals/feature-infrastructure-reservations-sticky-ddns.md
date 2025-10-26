# Feature Request: Infrastructure Reservations & Sticky DDNS (Delete‑on‑RELEASE vs EXPIRY)

## Summary

Introduce sensible defaults and controls so infrastructure hostnames remain stable while dynamic hygiene is preserved:
- Global DDNS behavior toggles: Delete‑on‑RELEASE (on), Delete‑on‑EXPIRY (off) with a grace window
- Reservation‑level Infrastructure mode: Sticky DNS, Pin Hostname, TTL class, Conflict Guard
- Provenance & auditability: record origin tags, show event cause, promote dynamic → static

## Problem

Critical hosts (LXCs, NAS, tuners) often use DHCP reservations but need stable DNS. Today, delete‑on‑EXPIRY can remove A/AAAA during brief lapses/renews or identifier drift, leaving CNAMEs dangling and breaking services.

## Proposal

1) Global DDNS behavior toggles
- Delete A/AAAA on lease EXPIRY (default OFF)
- Delete A/AAAA on DHCP RELEASE (default ON)
- Grace window before any EXPIRY deletion (e.g., 24–48h)

2) Reservation‑level “Infrastructure” mode
- Sticky DNS: never delete on EXPIRY; update only when reservation changes
- Pin Hostname: ignore client‑supplied names; publish reservation’s hostname
- TTL Class: Infrastructure (e.g., 86400s)
- Conflict Guard: refuse DDNS overwrite from a different MAC unless approved

3) Provenance & auditability
- Record tags: `static(admin)`, `ddns(dynamic)`, `ddns(reservation)`
- Audit: indicate RENEW/RELEASE/EXPIRY that caused changes
- Action: “Promote dynamic A → static”

## Benefits
- Eliminates NXDOMAIN flaps for infrastructure names
- Keeps dynamic environments clean via RELEASE (and optional EXPIRY + grace)
- Improves troubleshooting with clear record origin and event causes

## Workarounds today
- Use admin static A/AAAA and true static IPs outside DHCP
- Disable delete‑on‑expiry (hygiene trade‑offs) or avoid DDNS for infra altogether

## Environment
- Technitium DNS/DHCP 10.x
- Typical: Proxmox LXC reservation + Docker services, HDHomeRun tuner, Jellyfin/Dispatcharr by hostname

## Recommended Defaults (vendor chooses final)
- Delete‑on‑RELEASE = ON; Delete‑on‑EXPIRY = OFF; Grace window = ON (24–48h)
- Infrastructure preset: Sticky DNS, Pin Hostname, TTL=86400, Conflict Guard = ON
- Auto‑promote to static on reservation save = ON (see related DNS‑first brief)

## Related
- Overview/bundles: `../proposals-overview.md`
- Core behavior: `../infra-ddns.md`
- UX: `../reservations-ux.md`
- DNS‑first promotion: `./feature-dns-first-promotion.md`
- Retention summary: `./feature-dns-retention-summary.md`
- Technical analysis: `./feature-dns-retention-technical.md`
