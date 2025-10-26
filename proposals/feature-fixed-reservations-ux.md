# Feature Request: Dedicated Fixed Reservations Tab (separate from Active Leases)

## Summary

Add a first-class Fixed Reservations tab under DHCP to clearly separate user-designated reservations from dynamic clients. Provide fast CRUD, bulk actions, search/filters, conflict badges, and a one-click "promote lease → reservation" flow.

## Problem

Today, reservations management is often buried under scope pages or mixed with dynamic leases, which causes operator friction and conceptual confusion. Fixed reservations represent intentional, administrator-owned bindings; dynamic leases are ephemeral. The UI should reflect that distinction.

## Proposal

Navigation: `Dashboard · DNS · DHCP · Scopes · Fixed Reservations · Active Leases · Logs · Settings`

Fixed Reservations table:
- Columns: Hostname | IP | MAC | Scope | Infra | DDNS Policy | TTL Class | Updated | Actions
- Filters: Scope, Infra (yes/no), DDNS policy (Release/Expiry+grace), TTL class
- Views: All · Infra · Non‑Infra · Stale (no recent lease) · Conflicts (MAC mismatch)
- Badges: Infra, Pinned Hostname, Conflict (unexpected MAC), Stale
- Row actions: Edit · Duplicate · Disable · Delete
- Toolbar: Add · Delete · Bulk Actions · Export/Import · Search
- Empty-state: “No fixed reservations yet. Add your first reservation to pin a device to a stable name and IP.”

Drawer form:
- Hostname, MAC, IP (or auto), Scope, Notes
- DDNS: Register hostname; Delete on RELEASE/EXPIRY (grace hours); Allow client to change hostname
- Infrastructure preset: Sticky DNS; Pin Hostname; TTL=86400; Conflict Guard
- Buttons: Save · Cancel · Promote to Static A/AAAA · Test

Active Leases list (read-only):
- Columns: Hostname | IP | MAC | Scope | Client ID | Expires In | Last Seen
- Action: View · Create Reservation (pre-filled)

## Expected Behavior

- Reservations are managed in a dedicated tab separate from Active Leases
- One-click promotion from a lease to a fixed reservation
- Infra flags persisted and visible as badges/filters
- Export/Import retains TTL/Infra/DDNS policy fields
 - On creating or toggling a reservation, the system auto‑promotes any existing A/AAAA to admin static, pins hostname, and preserves PTR (with an option to opt out).

## Rationale

This eliminates conceptual conflation of dynamic clients with administrator-owned reservations, reduces operator error, and makes the infrastructure intent visible.

## Related

- UX spec: `reservations-ux.md`
- Core behavior: `infra-ddns.md`
- Overview/bundles: `proposals-overview.md`
