# Glossary — Terms used in this repo

This project uses precise language to avoid conflating DHCP concepts with DNS concepts.

## Core networking terms

- DHCP lease (dynamic client): A temporary address assignment issued to a client with no fixed reservation. Changes over time. Short TTLs are typical for any DNS names registered from such leases.

- DHCP fixed reservation (user-designated): An administrator-defined binding of a device identifier (MAC and/or Client ID/DUID) to a specific IP in a scope. Intended for stable, infrastructure-like devices (e.g., servers, tuners, controllers). Sometimes called a “static lease,” but this repo prefers “fixed reservation.”

- DNS record: An entry in a DNS zone, e.g., A/AAAA for forward mapping and PTR for reverse mapping.

## How DNS records are created

- DHCP‑generated DNS record (DDNS): A DNS record created/updated by the DHCP service as part of dynamic DNS update (DDNS). In this repo we tag their origin conceptually as `ddns(dynamic)` for dynamic leases and `ddns(reservation)` for fixed reservations.

- Admin‑created static DNS record: A DNS record created manually by an operator (not via DHCP update). Tagged conceptually as `static(admin)`.

## Policies and options

- Delete‑on‑RELEASE: Cleanup behavior that deletes DDNS records when the client actively releases its lease. Recommended default.

- Delete‑on‑EXPIRY: Cleanup behavior that deletes DDNS records when a lease times out. Risky for infrastructure; we recommend it be disabled by default and/or guarded by a grace window.

- Grace window: A delay (e.g., 24–48h) before performing expiry‑based cleanup, allowing normal churn without making names disappear.

- Infrastructure mode (reservation‑level): A reservation policy bundle that provides sticky DNS (no delete on expiry), pinned hostname, longer TTLs, and conflict guard.

## Identity nuances

- Client Identifier (DUID): An identifier many OSes use in addition to MAC. When VMs/containers are rebuilt, the DUID often changes; systems must not treat this as justification to remove the reservation’s DNS name.

## Terminology rules used here

- “DHCP records” is ambiguous. We avoid it. We say “DHCP‑generated DNS records (DDNS)” for DNS entries made by DHCP, and “admin‑created static DNS records” for manual entries.
- We distinguish clearly between “dynamic lease” (no reservation) and “fixed reservation” (user‑designated). Policies may (and should) differ for each.
