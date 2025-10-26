# Feature Request: Preserve DNS A/AAAA Records on DHCP Lease Expiry

## Summary

Add a per-scope option to retain A/AAAA records when a lease expires. This prevents accidental removal of infrastructure hostnames (on reservations) during routine churn while keeping dynamic hygiene via delete-on-RELEASE.

> Proposed checkbox: ☑ Do not delete DNS A/AAAA records on lease expiry

## Problem

With DHCP “Enable DNS Update” enabled, Technitium creates A/AAAA and PTR on lease grant and deletes them on lease expiry. That tight coupling removes even admin-created A records for reserved hosts when a lease briefly lapses or client identifiers change (e.g., VM rebuilds), causing NXDOMAIN and cascading service failures.

## Proposal

- Per-scope checkbox: “Do not delete DNS A/AAAA on lease expiry”
- Default posture: Delete-on-RELEASE = enabled; Delete-on-EXPIRY = disabled
- Optional: PTR cleanup controlled separately

## Expected behavior

- DHCP continues creating/updating DNS as today
- On lease expiry, A/AAAA records are retained unless explicitly removed by an admin
- PTR cleanup follows its own toggle

## Benefits

- Infrastructure hostnames remain stable; dependent services stop breaking
- Mixed environments (reservations + dynamic clients) become easier to operate
- No need for awkward static-IP carve-outs just to protect names

## Notes

- Complements broader “Infrastructure mode” for reservations (sticky DNS, pin hostname, TTL class, conflict guard)
- Works alongside a grace window for expiry scavenging if implemented

## Related

- Technical analysis: [DHCP/DNS Ownership Conflict](./feature-dns-retention-technical.md)
- Overview and bundles: [Proposals Overview](../proposals-overview.md)
