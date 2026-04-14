# Stations Affected

## 1. Scope

This document specifies how MTA encodes station-specific and direction-specific impacts in GTFS Realtime Service Alerts for subway feeds.

This document supplements the official GTFS Realtime specification. Where this document is silent, consumers and producers should defer to the canonical [GTFS Realtime specification](https://gtfs.org/documentation/realtime/reference/), including the definition of `EntitySelector`.

## 2. Purpose

The Stations Affected representation enables developers to identify the specific stations and travel directions directly impacted by a service condition.

This representation is intended to support more precise parsing, display, and rider messaging for Planned Work where station-level impact detail is available.

## 3. Data Model

### 3.1 Alert Entity

Stations Affected information is conveyed within the GTFS Realtime `alert.informed_entity` field.

Each element of `informed_entity` should conform to the GTFS Realtime `EntitySelector` type described in the official [GTFS Realtime specification](https://gtfs.org/documentation/realtime/reference/#message-entityselector).

This document focuses on the primary GTFS Realtime `EntitySelector` fields currently used for this representation. Other standard GTFS Realtime `EntitySelector` fields not described here, such as `trip`, may be used more broadly in future implementations.

### 3.2 Station-Level Representation

When station-level impact detail is provided, each directly affected station should be represented as a separate `informed_entity` entry.

When multiple stations are affected by the same alert, the feed should include multiple `informed_entity` entries rather than a single aggregated station selector.

### 3.3 Direction-Level Representation

When an alert applies to a specific direction of travel at a station, the corresponding `informed_entity` entry may include `direction_id`.

When an alert applies to both directions at a station, `direction_id` may be omitted.

## 4. Field Definitions

The following fields are the primary fields relevant to the current Stations Affected representation.

| Field | Type | Cardinality | Definition |
| --- | --- | --- | --- |
| `agency_id` | string | required | Identifies the agency associated with the affected service. |
| `route_id` | string | required | Identifies the route associated with the affected service. |
| `stop_id` | string | conditional | Identifies the directly affected station when station-level impact detail is provided. |
| `direction_id` | integer | conditional | Identifies the affected direction of travel when the impact is direction-specific. |

Additional GTFS Realtime `EntitySelector` fields that are not enumerated below may still apply to the same `informed_entity`, including standard fields such as `trip`. Consumers should continue to parse supported GTFS Realtime selector fields even when they are not the focus of this document.

### 4.1 `agency_id`

`agency_id` should identify the agency associated with the affected service.

Consumers should treat `agency_id` as part of the alert context for the relevant `informed_entity` entries.

### 4.2 `route_id`

`route_id` should identify the route associated with the affected service.

Consumers should treat `route_id` as part of the primary context for the relevant `informed_entity` entries when it is present.

### 4.3 `stop_id`

`stop_id` should contain a valid GTFS stop identifier for the affected subway station when station-level impact detail is provided.

Consumers should interpret the presence of `stop_id` as indicating that the specified station is directly affected by the alert.

### 4.4 `direction_id`

`direction_id` should use the following value mapping:

| Value | Meaning |
| --- | --- |
| `0` | Northbound |
| `1` | Southbound |

If `direction_id` is omitted on an `informed_entity` that includes `stop_id`, consumers should interpret the affected station as impacted in both directions.

## 5. Processing Rules

### 5.1 Producer Requirements

When station-level impact detail is provided, producers should emit one `informed_entity` entry per directly affected station.

Producers should include `agency_id` and `route_id` as part of the normal context for relevant `informed_entity` entries.

Producers may include additional standard or extension fields on the same `informed_entity` entry, provided those fields do not contradict the station-level meaning defined in this document.

### 5.2 Consumer Requirements

Consumers should evaluate each `informed_entity` entry independently.

Consumers should not infer that all stations on a route or route segment are affected solely because one or more station-specific `informed_entity` entries are present.

Consumers should prioritize station-specific entries in rider-facing messaging when presenting impacts from Planned Work or similar disruptions.

Consumers should use `direction_id` to distinguish northbound-only, southbound-only, and bidirectional impacts when station-level impact detail is present.

## 6. Example

### 6.1 Informative Example

The following example shows an alert in which three southbound stations on route `7` are directly affected.

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

### 6.2 Example Interpretation

In this example, the alert applies to stations `711`, `712`, and `713` on route `7`.

Because `direction_id` is `1` on each entry, the alert applies only to the southbound direction at each listed station.

## 7. Developer Notes

Stations Affected is an optional part of how MTA prepares and publishes alerts. Consumers should not assume that every alert will include Stations Affected data. Stations Affected data will be tagged when a service change is considered significant enough to warrant that additional level of detail.

This representation is initially introduced for planned service changes, and may be extended to unplanned service changes in the future.
