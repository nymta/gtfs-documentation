# GTFS-RT Update: Status Priority in Alert Entity ID

## Background: Alert Statuses in the Content Management System

Service alerts and planned work for all agencies — NYCT Subway, NYCT Bus, Long Island Rail Road (LIRR), and Metro-North Railroad (MNR) — are created and managed in a shared content management system (CMS). Every alert created in the CMS is assigned a **status**, which describes the nature of the service impact (e.g., a delay, a reroute, a schedule change, etc.).

There are **35 distinct statuses** used across all agencies and modes. Because the same list is shared across agencies, some statuses may appear similar or redundant at first glance (for example, both "Detour" and "Planned - Detour" exist), but each is intentionally distinct and may be applied differently depending on the agency and whether the alert reflects planned or unplanned service impacts.

Each status has an associated **rank**, which indicates its relative priority. Ranks range from **1 (lowest priority) to 35 (highest priority)**. When an alert could reasonably be described by more than one status, the higher-ranked status takes precedence.

### Status Rank Table

| Rank | Status |
|------|--------|
| 1 | No Scheduled Service |
| 2 | Information Outage |
| 3 | Station Notice |
| 4 | Special Notice |
| 5 | Weekday Schedule |
| 6 | Weekend Schedule |
| 7 | Saturday Schedule |
| 8 | Sunday Schedule |
| 9 | Extra Service |
| 10 | Boarding Change |
| 11 | Special Schedule |
| 12 | Expect Delays |
| 13 | Reduced Service |
| 14 | Planned - Express to Local |
| 15 | Planned - Extra Transfer |
| 16 | Planned - Stops Skipped |
| 17 | Planned - Detour |
| 18 | Planned - Reroute |
| 19 | Planned - Substitute Buses |
| 20 | Planned - Part Suspended |
| 21 | Planned - Suspended |
| 22 | Service Change |
| 23 | Planned Work |
| 24 | Some Delays |
| 25 | Express to Local |
| 26 | Delays |
| 27 | Cancellations |
| 28 | Delays and Cancellations |
| 29 | Stops Skipped |
| 30 | Severe Delays |
| 31 | Detour |
| 32 | Reroute |
| 33 | Substitute Buses |
| 34 | Part Suspended |
| 35 | Suspended |

---

## GTFS-RT Update: Status Priority Appended to Entity ID

### Summary

This update applies to the GTFS-RT feeds for **all agencies** (NYCT Subway, NYCT Bus, LIRR, and MNR). The `id` field under `entity` in the GTFS-RT feed will now include the **status rank** of the alert, appended to the existing alert ID and separated by a colon (`:`).

This allows consumers of the feed to identify the priority of an alert's status directly from the entity ID, without needing to cross-reference the status separately.

### Format

```
entity.id = "<existing alert id>:<status rank>"
```

### Example

**Before:**
```json
"entity": [{"id": "lmm:alert:532243", ...}]
```

**After:**
```json
"entity": [{"id": "lmm:alert:532243:31", ...}]
```

In this example, the appended value `31` corresponds to the **Detour** status, per the rank table above.

### Notes for Consumers

- The status rank appended to the `id` reflects the rank at the time the feed entry was generated. If an alert's status changes, the rank suffix will update accordingly on the next feed refresh.
- The rank is purely additive to the existing ID structure — no other part of the `id` format changes, and existing ID prefixes remain stable.
- Because ranks are shared across all agencies, the same numeric suffix (e.g., `:31`) will always correspond to the same status (e.g., Detour) regardless of agency.
- This change applies uniformly across all four agency feeds; no agency-specific exceptions apply.
