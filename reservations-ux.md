# Fixed Reservations UX Proposal

## Goals
- Make adding/editing a reservation a **single, light action**—not a full-page scope save.
- Give operators search, filters, and bulk actions for dozens/hundreds of entries.

## Navigation
Left nav: `Dashboard · DNS · DHCP · Scopes · **Fixed Reservations** · **Active Leases** · Logs · Settings`

## Fixed Reservations Table
Columns: **Hostname | IP | MAC | Scope | Infra | DDNS Policy | TTL Class | Updated | Actions**

Toolbar: **Add · Delete · Bulk Actions · Export/Import · Search**

Empty‑state copy:
- “No fixed reservations yet. Add your first reservation to pin a device to a stable name and IP.”

Badges:
- Infra (blue), Dynamic (gray)
- DDNS: Release / Expiry+24h

Filters: Scope, Infra, DDNS policy, TTL class
Views:
- All · Infra · Non‑Infra · Stale (no recent lease) · Conflicts (MAC mismatch)

Row actions: Edit · Duplicate · Disable · Delete
Row indicators:
- Badge when hostname is pinned; warning when client‑reported hostname disagrees
- Conflict badge when DDNS update comes from unexpected MAC

## Drawer Form (Add/Edit)
Fields:
- Hostname, MAC, IP (or auto), Scope, Notes

DDNS (for this reservation):
- [x] Register hostname in DNS
- Delete record on: (•) RELEASE   ( ) EXPIRY   Grace: [24] h
- [ ] Allow client to change hostname

Infrastructure:
- [x] Treat as **Infrastructure** (Sticky DNS; Pin Hostname; TTL=86400; Conflict Guard)

Auto‑promotion (default on):
- [x] Promote current A/AAAA (if any) to admin static on save; pin hostname; preserve PTR

Buttons: **Save · Cancel · Promote to Static A/AAAA · Test**

## DNS View integration (admin-created A/AAAA → reservation)
In the DNS Records UI, when an admin creates or selects a static A/AAAA:
- Show banner if IP is inside a DHCP scope and not reserved: “This IP is in DHCP. Create a Fixed Reservation?” [Create] [Dismiss]
- If the IP is leased dynamically: “Dynamic lease detected. Reserve now to prevent reallocation.” [Create Reservation]
- If MAC/Client ID unknown: “Watching for next lease on this IP to bind reservation.”
- Link back to the pre‑filled Fixed Reservation drawer.

Settings hint:
- This flow follows Global Settings → DNS/DHCP integration:
	- Prompt vs Auto‑create vs Ignore
	- Drift alerts: Warn vs Block
	- Auto‑watch behavior

## Active Leases (read‑only list)
Columns: **Hostname | IP | MAC | Scope | Client ID | Expires In | Last Seen**
Actions: View · Create Reservation (pre‑filled)

Notes:
- This list is distinct from Fixed Reservations to avoid conflating dynamic clients with user‑designated infrastructure.

## Scope Page Cleanup
- Remove giant reservations table; show a small counter + deep link: “Reservations (N) →”.

## Acceptance Criteria
- Dedicated Fixed Reservations tab exists, separate from Active Leases.
- CRUD on reservations without navigating away from the list.
- Per‑reservation infra flags persisted and reflected as badges/filters.
- Export/import preserves infra/TTL/policy fields.
- From an Active Lease, an operator can one‑click “Create Reservation” to promote a device.
 - Saving a new reservation auto‑promotes any existing A/AAAA to admin static and pins hostname (unless the operator unchecks the option).
- From DNS view, creating a static A/AAAA offers to create a fixed reservation (or auto‑creates per policy) and sets drift alerts if declined.
