# Documentation Index

Welcome! This folder aggregates the design notes and proposals for infrastructure‑friendly DNS/DHCP behavior and UX.

## Contents

- Future direction and principles
  - [Future Direction: Intention‑Aware, Self‑Hardening DNS/DHCP](./future-direction.md)
- Core specs
  - [Infrastructure‑Level DDNS Behavior](./infra-ddns.md)
  - [Reservations UX Proposal](./reservations-ux.md)
- Product/process notes
  - [Prioritization & UX‑as‑Function Rubric](./prioritization.md)
- Proposals (vendor‑facing write‑ups)
  - [Proposals Overview and Implementation Options](./proposals-overview.md) ← start here for a one‑page summary
  - [Feature Request: Preserve DNS A/AAAA on Lease Expiry](./proposals/feature-request.md)
  - [Technical Analysis: DHCP/DNS Record Ownership Conflict](./proposals/feature-dns-retention-technical.md)
  - [Summary: Delete‑on‑RELEASE vs Delete‑on‑EXPIRY + Infrastructure Mode](./proposals/feature-dns-retention-summary.md)

## Related documents

- [Project README](../README.md)
- [Roadmap](../ROADMAP.md)
- [Changelog](../CHANGELOG.md)
- [Contributing](../CONTRIBUTING.md)

## Conventions

- Example zone: `localdomain`
- TTLs in seconds (infra default: 86400)
- MAC addresses uppercase, colon‑delimited
