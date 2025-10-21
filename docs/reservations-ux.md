# Reservations UX Proposal

## Goals
- Make adding/editing a reservation a **single, light action**—not a full-page scope save.
- Give operators search, filters, and bulk actions for dozens/hundreds of entries.

## Navigation
Left nav: `Dashboard · DNS · DHCP · Scopes · **Reservations** · Logs · Settings`

## Reservations Table
Columns: **Hostname | IP | MAC | Scope | Infra | DDNS Policy | TTL Class | Updated | Actions**

Toolbar: **Add · Delete · Bulk Actions · Export/Import · Search**

Badges:
- Infra (blue), Dynamic (gray)
- DDNS: Release / Expiry+24h

Filters: Scope, Infra, DDNS policy, TTL class

Row actions: Edit · Duplicate · Disable · Delete

## Drawer Form (Add/Edit)
Fields:
- Hostname, MAC, IP (or auto), Scope, Notes

DDNS:
- [x] Register hostname in DNS
- Delete record on: (•) RELEASE   ( ) EXPIRY   Grace: [24] h
- [ ] Allow client to change hostname

Infrastructure:
- [x] Treat as **Infrastructure** (Sticky DNS; Pin Hostname; TTL=86400; Conflict Guard)

Buttons: **Save · Cancel · Promote to Static A/AAAA · Test**

## Scope Page Cleanup
- Remove giant reservations table; show a small counter + deep link: “Reservations (N) →”.

## Acceptance Criteria
- CRUD on reservations without navigating away from the list.
- Per-reservation infra flags persisted and reflected as badges/filters.
- Export/import preserves infra/TTL/policy fields.
