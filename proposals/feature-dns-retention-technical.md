# Technical Analysis: DHCP/DNS Record Ownership Conflict in Technitium

## Overview

With DHCP “Enable DNS Update,” Technitium creates A/AAAA and PTR on lease grant and deletes them on lease expiry. This tight coupling unintentionally removes admin-created or reservation-intended A/AAAA records during brief lapses or identifier changes, causing avoidable outages.

## Root cause

1. DHCP treats itself as authoritative for records in-scope when DNS update is enabled.
2. Lease-centric cleanup deletes A/AAAA on expiry, including for reservations.
3. No distinction between admin-created vs DHCP-created records during cleanup.

### Example

- Admin creates `my-server.lan -> 192.168.1.50`.
- Reservation exists for 192.168.1.50 (MAC X).
- Host goes offline past lease; cleanup deletes A/AAAA despite admin intent.

## Client identifier drift

Rebuilt VMs/LXCs often change Client ID (DUID). The old lease expires and triggers deletion while a new lease for the same reservation is already active, flapping DNS.

## Workaround today (not ideal)

Use static IPs outside the pool and manual DNS. This avoids DHCP ownership but loses the benefits of reservations and automation.

## Proposed enhancement

- Per-scope checkbox: “Do not delete DNS A/AAAA on lease expiry”
- Default posture: Delete-on-RELEASE = enabled; Delete-on-EXPIRY = disabled
- PTR cleanup governed separately

## Expected behavior

- Skip A/AAAA deletion during lease-expiry cleanup when the flag is set
- Keep creating/updating records as today
- PTR logic controlled by its own toggle

## Optional safeguards

- Don’t delete records whose creation predates DHCP association
- Maintain origin tags: manual vs DHCP-created vs from-reservation
- Per-scope setting for fine-grained control

## Implementation sketch

Add a conditional check in the lease-expiry handler (cleanup task) to bypass A/AAAA deletion when the scope’s retain-on-expiry flag is true. Persist origin metadata in DNS records to enable provenance-aware behavior.

## Impact

Significant stability improvements for mixed environments (reservations + dynamic clients) with minimal code surface and clear operator expectations.

## Related

- Summary request: [Preserve DNS on Lease Expiry](./feature-dns-retention-summary.md)
- Overview and bundles: [Proposals Overview](../proposals-overview.md)
