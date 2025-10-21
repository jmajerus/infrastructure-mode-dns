# Infrastructure-Level DDNS Behavior

## Problem
DDNS that deletes A/AAAA on **lease EXPIRY** causes critical names to vanish during brief lapses/renews/hostname churn. CNAMES dangle; services break.

## Objectives
- Infra names **never** disappear due to expiry.
- Dynamic clients still get cleaned up.
- Operators can see *why* a record changed.

## Proposed Behavior

### 1) Delete-on-RELEASE vs Delete-on-EXPIRY (Global & Per-Scope)
- **Default**: Delete A/AAAA on **RELEASE** (enabled); Delete on **EXPIRY** (disabled).
- If EXPIRY cleanup is enabled, allow a **grace window** (24–48h) before scavenging.

**UI copy (Global DHCP settings):**
- [ ] Delete A/AAAA on lease **EXPIRY** (aggressive cleanup; not for infrastructure)
- [x] Delete A/AAAA on **DHCP RELEASE** (recommended)
- Grace before EXPIRY delete: [ 24 ] hours

### 2) Reservation-Level “Infrastructure” Mode
Flags stored on the reservation:
- **Sticky DNS**: never delete on EXPIRY; update A/AAAA only when reservation changes.
- **Pin Hostname**: ignore client-reported hostname; publish reservation name.
- **TTL Class: Infrastructure**: default 86400s.
- **Conflict Guard**: refuse DDNS overwrite from different MAC; log event.

**UI (Reservation form):**
- [x] Treat as **Infrastructure** (Sticky DNS, Pin Hostname, TTL=86400s, Conflict Guard)
- [ ] Allow client to update hostname
- Delete on: (•) RELEASE  ( ) EXPIRY   Grace: [24] h

### 3) Provenance & Audit
- Record label in DNS UI: `static(admin)` | `ddns(dynamic)` | `ddns(reservation)`
- Audit log: “A updated (RENEW|RELEASE|EXPIRY) host=H mac=M ip=I scope=S ttl=NNN”
- Button: **Promote dynamic A → static**

### 4) API sketch
```json
GET /api/dhcp/reservations
POST /api/dhcp/reservations { "hostname": "epg", "mac": "AA:BB:CC:DD:EE:FF", "ip": "192.168.1.56", "scope": "LAN", "infra": true, "ddnsPolicy": {"deleteOn": "release", "graceHours": 24 }, "ttlClass": "infrastructure" }
PATCH /api/dhcp/reservations/:id
```

### Acceptance Criteria
- Infra reservations never lose A/AAAA on EXPIRY.
- Dynamic clients still clean on RELEASE (and optionally on EXPIRY if enabled).
- Provenance visible in UI and export; logs show event cause.

### Migration
- Keep existing installs as-is until toggles are set.
- First-run banner: “Mark DHCP reservations as **Infrastructure** for sticky DNS.”
