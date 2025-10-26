**Title:** Add “Delete-on-RELEASE vs Delete-on-EXPIRY” and an “Infrastructure” mode for DHCP reservations (Sticky DDNS)

**Problem**
Critical hosts (LXCs, NAS, tuners) are on DHCP reservations but need stable DNS. Today, delete-on-EXPIRY removes A/AAAA during brief lapses/renews, leaving CNAMEs dangling and breaking services.

**Requested features**
1. Global DDNS behavior toggles:
   - Delete A/AAAA on lease EXPIRY (off by default)
   - Delete A/AAAA on DHCP RELEASE (on by default)

2. Reservation-level “Infrastructure” mode with:
   - Sticky DNS: never delete on EXPIRY; update only when reservation changes
   - Pin Hostname: ignore client-supplied names; publish reservation’s hostname
   - TTL Class: Infrastructure (e.g., 12–24h)
   - Conflict Guard: refuse DDNS overwrite from a different MAC unless approved

3. Grace window before any EXPIRY deletion (e.g., 24–48h). Optionally check no ARP or no DNS queries before scavenging.

4. Provenance & auditability
   - Record tags: `static(admin)`, `ddns(dynamic)`, `ddns(reservation)`
   - Audit log: indicate RENEW/RELEASE/EXPIRY that caused the change
   - Action: “Promote dynamic A → static”

**Default behavior (proposed)**
- Global: Delete-on-RELEASE enabled; Delete-on-EXPIRY disabled
- Reservations marked Infrastructure: Sticky DNS + TTL=86400s
- Dynamic scopes: Delete-on-EXPIRY enabled with short TTLs

**Benefits**
- No more NXDOMAIN flaps for infrastructure names
- Dynamic hygiene preserved for roaming clients
- Clearer troubleshooting with provenance and logs

**Workarounds today**
- Create static A records in a Primary Zone for infra hosts; use DHCP only for leases
- Disable delete-on-expiry (hygiene trade-offs) or avoid DDNS for infra altogether

**Environment**
- Technitium DNS/DHCP 10.x
- Typical use: Proxmox LXC reservation + Docker services, HDHomeRun tuner, Jellyfin/Dispatcharr by hostname

   - **TTL Class: Infrastructure** (e.g., 12–24h).   - **Sticky DNS:** never delete on EXPIRY; update only when reservation changes.

   - **Conflict Guard:** refuse DDNS overwrite from a different MAC unless approved.   - **Pin Hostname:** ignore client-supplied names; publish reservation’s hostname.

   - **TTL Class: Infrastructure** (e.g., 12–24h).

3. **Grace window** before any EXPIRY deletion (e.g., 24–48h), optionally check no ARP or no DNS queries before scavenging.   - **Conflict Guard:** refuse DDNS overwrite from a different MAC unless approved.



4. **Provenance & Auditability**3. **Grace window** before any EXPIRY deletion (e.g., 24–48h), optionally check no ARP or no DNS queries before scavenging.

   - Record tags: `static(admin)`, `ddns(dynamic)`, `ddns(reservation)`.

   - Audit log entries: indicate RENEW/RELEASE/EXPIRY that caused the change.4. **Provenance & Auditability**

   - Button: “Promote dynamic A → static”.   - Record tags: `static(admin)`, `ddns(dynamic)`, `ddns(reservation)`.

   - Audit log entries: indicate RENEW/RELEASE/EXPIRY that caused the change.

**Default behavior (proposed)**   - Button: “Promote dynamic A → static”.

- Global: Delete-on-RELEASE **enabled**; Delete-on-EXPIRY **disabled**.

- Reservations marked Infrastructure: Sticky DNS + TTL=86400s.**Default behavior (proposed)**

- Dynamic scopes: Delete-on-EXPIRY **enabled**, short TTLs.- Global: Delete-on-RELEASE **enabled**; Delete-on-EXPIRY **disabled**.

- Reservations marked Infrastructure: Sticky DNS + TTL=86400s.

**Benefits**- Dynamic scopes: Delete-on-EXPIRY **enabled**, short TTLs.

- No more NXDOMAIN flaps for infrastructure names.

- Dynamic hygiene preserved for roaming clients.**Benefits**

- Clearer troubleshooting with provenance and logs.- No more NXDOMAIN flaps for infrastructure names.

- Dynamic hygiene preserved for roaming clients.

**Workarounds today**- Clearer troubleshooting with provenance and logs.

- Create static A records in a Primary Zone for infra hosts; rely on DHCP only for leases.

- Disable delete-on-expiry (with hygiene trade-offs) or avoid DDNS for infra altogether.**Workarounds today**

- Create static A records in a Primary Zone for infra hosts; rely on DHCP only for leases.

**Environment**- Disable delete-on-expiry (with hygiene trade-offs) or avoid DDNS for infra altogether.

- Technitium DNS/DHCP 10.x

- Typical use: Proxmox LXC (reservation) + Docker services, HDHomeRun tuner, Jellyfin/Dispatcharr pulling by hostname.**Environment**

- Technitium DNS/DHCP 10.x
- Typical use: Proxmox LXC (reservation) + Docker services, HDHomeRun tuner, Jellyfin/Dispatcharr pulling by hostname.
