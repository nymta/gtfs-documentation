# Stations Affected

## Overview

Stations Affected is an MTA enhancement that adds station-level and direction-specific detail to GTFS-Realtime service alerts. It is designed to improve how Planned Work and other alerting scenarios identify the exact stations and travel directions impacted by a disruption.

This feature is being introduced first for NYCT Planned Work, with support for additional NYCT alert types expected to follow. Consumers should expect more granular affected-station data in the GTFS-RT alerts feed as this rollout expands.

## GTFS-RT Representation

When Stations Affected is present on an alert, the GTFS-RT `alert` entity includes station and directional information in `informed_entity`. MTA uses the standard GTFS-Realtime `EntitySelector` structure documented in the official [GTFS Realtime specification](https://gtfs.org/documentation/realtime/feed-entities/service-alerts/#entityselector).

Each affected station is represented as its own `informed_entity` entry.

### Fields Used in `informed_entity`

| Field | Type | Description |
| --- | --- | --- |
| `stop_id` | string | Identifies a directly affected station. |
| `direction_id` | integer | Identifies the travel direction affected by the alert. |

### `direction_id` Values

| Value | Meaning |
| --- | --- |
| `0` | Northbound |
| `1` | Southbound |
| omitted | Both directions |

If `direction_id` is omitted, consumers should interpret the alert as applying to both directions at that station.

## Repeated Station Entities

Affected stations are represented by repeated `informed_entity` entries. Each directly impacted station appears separately, even when multiple stations share the same route, direction, and alert context.

Consumers should not assume a single `informed_entity` entry applies to an entire corridor or route segment when `stop_id` values are provided. Instead, each listed station should be treated as explicitly affected.

## Example

The example below shows three southbound stations on route `7` represented as directly affected by a Planned Work alert.

```json
"informed_entity": [
  {
    "agency_id": "MTASBWY",
    "route_id": "7",
    "stop_id": "711",
    "direction_id": 1,
    "transit_realtime.mercury_entity_selector": {
      "sort_order": "MTASBWY:7:3"
    }
  },
  {
    "agency_id": "MTASBWY",
    "route_id": "7",
    "stop_id": "712",
    "direction_id": 1,
    "transit_realtime.mercury_entity_selector": {
      "sort_order": "MTASBWY:7:3"
    }
  },
  {
    "agency_id": "MTASBWY",
    "route_id": "7",
    "stop_id": "713",
    "direction_id": 1,
    "transit_realtime.mercury_entity_selector": {
      "sort_order": "MTASBWY:7:3"
    }
  }
]
```

## Consumer Guidance

- Treat each `informed_entity` with a `stop_id` as a station-specific impact.
- Use `direction_id` to determine whether the impact is northbound only, southbound only, or both directions when omitted.
- Prioritize affected stations in rider-facing messaging when presenting Planned Work or disruption details.
