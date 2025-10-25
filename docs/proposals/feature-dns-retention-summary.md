# Feature Request: Preserve DNS A/AAAA Records on DHCP Lease Expiry

## Summary

Technitium DNS Server currently deletes DNS A/AAAA records when a DHCP lease expires if **Enable DNS Update** is enabled. This behavior is tightly coupled—creation and deletion are handled as one inseparable process. Unfortunately, it causes manual DNS records, even for reserved hosts, to be deleted when their leases expire.

This results in disruption to semi-static infrastructure (e.g., Proxmox nodes, Home Assistant, Zigbee2MQTT, or other self-hosted systems) that depend on consistent hostname resolution.

## Problem

When *Enable DNS Update* is enabled under a DHCP scope, the DHCP server claims ownership of all DNS records within its managed range. Even manually created A/AAAA entries are deleted when a reservation's lease expires or when the client identifier changes (as often occurs with rebuilt VMs, LXCs, or reinstalled OSes).

This makes it impossible to rely on DHCP reservations for stable DNS name persistence—an essential need for home labs, small business networks, and mixed-static environments.

## Proposed Solution

Introduce a per-scope checkbox:

> **☑ Do not delete DNS A/AAAA records on lease expiry**

**Expected Behavior:**

* DHCP continues creating and updating DNS records normally.
* Upon lease expiry or deletion, DNS records are retained unless explicitly removed by an administrator.
* PTR cleanup may remain separate or optional.

## Benefits

* Prevents accidental loss of manually created or reserved host records.
* Simplifies DNS/DHCP management for mixed environments.
* Eliminates the need for awkward static-IP workarounds.
* Ensures reliable hostname persistence for critical home-lab and IoT devices.

## Example Use Case

A Proxmox node with a DHCP reservation (192.168.1.50) and manual DNS record (`pve1.localdomain`) goes offline for maintenance. After the reservation lease expires, the DHCP cleanup process removes its DNS entry—even though the record was manually created. A simple checkbox to retain DNS records on lease expiry would prevent this failure scenario entirely.

## Acknowledgements

This proposal draws on collaborative analysis from multiple AI-assisted sessions:

* **Google Gemini Pro (October 2025):** Clarified Technitium's internal record-ownership and lease-expiry behavior.
* **OpenAI ChatGPT (GPT-5, October 2025):** Synthesized the technical and practical implications into this documented feature request.

Both tools contributed to identifying the architectural limitation and articulating a balanced, implementable feature enhancement.
