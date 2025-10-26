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

Implicit invariant when creating/toggling a fixed reservation:
- **Auto‑promote DNS to static**: if an A/AAAA exists (DDNS or manual), convert it to an admin static record at save time, pin the hostname, and preserve PTR. Record provenance (who/when) and note prior origin (dynamic vs manual).

**UI (Reservation form):**
- [x] Treat as **Infrastructure** (Sticky DNS, Pin Hostname, TTL=86400s, Conflict Guard)
- [ ] Allow client to update hostname
- Delete on: (•) RELEASE  ( ) EXPIRY   Grace: [24] h
- [x] Automatically promote current DNS A/AAAA to static on save (recommended)

### 3) Provenance & Audit
- Record label in DNS UI: `static(admin)` | `ddns(dynamic)` | `ddns(reservation)`
- Audit log: “A updated (RENEW|RELEASE|EXPIRY) host=H mac=M ip=I scope=S ttl=NNN”
- Button: **Promote dynamic A → static**

### 3a) DNS‑first promotion & guardrails (admin creates A/AAAA first)
When an operator manually creates an admin static A/AAAA:
- If the IP is inside a DHCP scope and not reserved, offer to create a **fixed reservation** for that IP (pre‑filled), or auto‑create if a policy toggle is on.
- If the IP is currently leased dynamically, flag the lease as “Should be reserved” and warn on reallocation attempts.
- If MAC/Client ID is unknown, add the IP to a **watchlist**; the next observed lease for that IP triggers a prompt to bind it.
- Record provenance linking the DNS record to the reservation creation or to the watchlist event.

Drift detection:
- Alert if DHCP attempts to reassign a watched IP that has an admin static A/AAAA.
- Alert if a dynamic DDNS update tries to overwrite a pinned hostname.

### 4) API sketch
```json
GET /api/dhcp/reservations
POST /api/dhcp/reservations { "hostname": "epg", "mac": "AA:BB:CC:DD:EE:FF", "ip": "192.168.1.56", "scope": "LAN", "infra": true, "ddnsPolicy": {"deleteOn": "release", "graceHours": 24 }, "ttlClass": "infrastructure" }
PATCH /api/dhcp/reservations/:id
```

### 5) Global DNS–DHCP reconciliation policy

Operator defaults to reduce gotchas and drift. Applies globally, with per‑scope overrides.

UI (Global Settings → DNS/DHCP integration):
- [x] When admin creates static A/AAAA for an IP inside a DHCP scope:
	- (•) Prompt to create a Fixed Reservation  ← recommended default
	- ( ) Auto‑create a Fixed Reservation
	- ( ) Ignore (no action)
- [x] Enable drift detection for watched IPs and pinned hostnames  ← recommended default
	- Alert severity: (•) Warn ( ) Block unless override  ← recommended default: Warn
- [x] Auto‑watch non‑reserved in‑scope static A/AAAA and notify on first lease seen  ← recommended default

API (settings):
```json
GET /api/settings/dnsDhcp
PATCH /api/settings/dnsDhcp {
	"onStaticDnsInScope": "prompt|auto|ignore",
	"driftDetection": { "enabled": true, "mode": "warn|block" },
	"autoWatch": true
}
```

### Acceptance Criteria
- Infra reservations never lose A/AAAA on EXPIRY.
- Dynamic clients still clean on RELEASE (and optionally on EXPIRY if enabled).
- Provenance visible in UI and export; logs show event cause.

### Other ways to create “infrastructure components”
Fixed reservations + auto‑promoted static DNS is the preferred, auditable path. If needed, operators can also:
- Use true static IPs outside the DHCP pool + admin‑created static A/AAAA (no DDNS ownership)
- Manage static assignments via IPAM and publish admin static DNS from there
- Manually promote/pin existing DDNS to static without creating a reservation

Trade‑offs: Alternatives bypass DHCP automation and provenance; choose them when DHCP isn’t authoritative for those subnets or when external IPAM governs addressing.

### Migration
- Keep existing installs as-is until toggles are set.
- First-run banner: “Mark DHCP reservations as **Infrastructure** for sticky DNS.”
