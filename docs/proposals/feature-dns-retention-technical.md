# Technical Analysis: DHCP/DNS Record Ownership Conflict in Technitium

## Overview

Technitium DNS Server’s *Enable DNS Update* option under DHCP scopes is designed to automate DNS record management—creating A/AAAA and PTR records when leases are granted, and deleting them upon lease expiry. However, this coupling of creation and deletion has an unintended side effect: it overrides administrator-created DNS records and removes them even when they correspond to reserved or semi-static hosts.

## Root Cause

The behavior results from Technitium’s current **record ownership model**:

1. **DHCP as Authoritative Owner:** When *Enable DNS Update* is active, DHCP assumes it is the source of truth for all DNS entries within its managed address range.
2. **Lease-Centric Cleanup:** When a lease expires (even for a reservation), the DHCP cleanup process deletes DNS records matching that IP.
3. **No Manual Override:** The cleanup logic does not differentiate between records created manually and those created by the DHCP service.

### Example Scenario

* Administrator creates a manual DNS record: `my-server.lan -> 192.168.1.50`.
* A DHCP reservation is added for the same IP and MAC address.
* The device goes offline beyond its lease time.
* The DHCP cleanup process removes the record on expiry, regardless of its manual creation.

This occurs even though the administrator intended for the DNS record to persist.

## Client Identifier Drift

Modern operating systems, VMs, and container environments often generate a **Client Identifier (DUID)** distinct from the MAC address. When a VM or container is rebuilt, its DUID changes. This leads to:

1. The DHCP server issuing a new lease (with a new Client ID) for the same reserved IP.
2. The old lease—tied to the old Client ID—expiring.
3. The cleanup process deleting DNS records associated with the expired lease, despite the new lease being active for the same IP.

This creates a frustrating cycle where DNS entries for otherwise stable devices continually vanish.

## Workaround (Inelegant)

Administrators must currently assign **true static IPs outside the DHCP pool** and manually create DNS entries. This removes the DHCP server’s ownership of that IP, preventing deletion. However, it negates the convenience and control offered by DHCP reservations and complicates automated provisioning.

## Proposed Enhancement

Introduce a DHCP-scope option:

> **☑ Do not delete DNS A/AAAA records on lease expiry**

### Expected Behavior

* DNS records created or managed by DHCP remain after lease expiry unless explicitly deleted by the administrator.
* PTR record cleanup could remain under a separate optional flag.
* Logic in the lease-cleanup routine would skip A/AAAA deletions when this flag is set.

### Optional Safeguards

* Skip deletion of any record whose creation timestamp predates its DHCP association.
* Maintain a distinct metadata tag in DNS entries denoting origin (manual vs. DHCP-created).
* Apply the setting per scope for fine-grained control.

## Implementation Notes

The relevant logic resides in Technitium’s lease expiry handler (possibly within the *LeaseExpiryTask* or equivalent cleanup thread). Modification would involve adding a conditional check for this new flag before performing DNS deletion operations.

## Impact

This small change yields outsized reliability improvements in hybrid environments where both dynamic and static infrastructure coexist. It enhances user trust and prevents unnecessary manual repair of DNS zones.

## Sources and Acknowledgements

* **Google Gemini Pro (October 2025):** Provided detailed behavioral analysis of the DHCP/DNS record-ownership conflict.
* **OpenAI ChatGPT (GPT-5, October 2025):** Structured and expanded the analysis to form this technical implementation proposal.
