# Prioritization & UX-as-Function Rubric


This note captures when UI changes rise to the level of **core capability**, plus a simple scoring rubric and milestones to help plan work.


## When the UI *is* the capability
- **High‑frequency tasks:** If admins touch it weekly/daily (e.g., add/edit reservations), UX quality directly equals system utility.
- **Error‑prone flows:** If misconfiguration is common (wrong MAC/hostname; dangling CNAMEs), UI validation and previews are reliability features.
- **Discoverability barriers:** A buried setting is functionally equivalent to not shipping it. Navigation and labeling become part of the feature.
- **Batch/scale needs:** Bulk actions, filters, and search turn a one‑off control into an operational tool.
- **Tight feedback loops:** Inline tests (e.g., **Test DNS**), live validation, and non‑destructive previews reduce downtime and incident rate.


## Scoring rubric (0–3 each)
1) **Uptime / Data integrity** — prevents outages (NXDOMAIN, stale/hijacked records).
2) **Task frequency & time saved** — minutes saved per week per admin.
3) **Error‑rate reduction** — misconfigs/support incidents avoided.
4) **Adoption unlock** — does this make the feature practically usable?
5) **Reversibility** — can we ship guardrails now and iterate safely?


> **Prioritize** items with high scores on 1–3, or that unlock adoption (4).


## Milestones (proposal)
- **v0.1 Core** — *Behavior + minimal UI*
- Delete‑on‑**RELEASE** vs **EXPIRY** + **grace window** (backend spec)
- Global & per‑scope toggles in UI (so it’s actually usable Day 1)
- **v0.2 Reservations Tab** — *Operational UX*
- Standalone tab, searchable table, filters, drawer edit, badges, bulk actions
- **v0.3 Provenance & Audit** — *Debuggability*
- Record tags (`static(admin)`, `ddns(dynamic)`, `ddns(reservation)`), audit events (RENEW/RELEASE/EXPIRY), “Promote dynamic → static”
- **v0.4 Infra Presets** — *One‑click safety*
- **Treat as Infrastructure** preset: Sticky DNS, Pin Hostname, TTL class, Conflict guard


## Success metrics
- **p50/p95 time‑to‑add** reservation
- **Misconfig rate** (MAC/name conflicts, missing A)
- **NXDOMAIN flaps/month** for infra names
- **Task success**: new admin adds 5 reservations error‑free in <3 minutes


## Notes
- Keep examples generic (e.g., `localdomain`, `192.168.1.0/24`).
- Prefer longer TTLs for infra (e.g., 86400s) and short TTLs for dynamic clients.
- Avoid mixing `.local` (mDNS) with unicast zones; publish DHCP Option 6 and 15 consistently.