# Stations Affected

## 1. Scope

This document specifies how MTA encodes station-specific and direction-specific impacts in GTFS-Realtime Service Alerts for subway feeds.

This document supplements the official GTFS-Realtime specification. Where this document is silent, consumers and producers should defer to the canonical GTFS-Realtime specification, including the definition of `EntitySelector`.

## 2. Purpose

The Stations Affected representation enables an alert to identify the specific stations and travel directions directly impacted by a service condition.

This representation is introduced to support more granular communication of Planned Work and may be used for additional subway alert types as MTA expands implementation.

## 3. Data Model

### 3.1 Alert Entity

Stations Affected information is conveyed within the GTFS-Realtime `alert.informed_entity` field.

Each element of `informed_entity` should conform to the GTFS-Realtime `EntitySelector` type described in the official [GTFS Realtime specification](https://gtfs.org/documentation/realtime/feed-entities/service-alerts/#entityselector).

### 3.2 Station-Level Representation

Each directly affected station should be represented as a separate `informed_entity` entry.

When multiple stations are affected by the same alert, the feed should include multiple `informed_entity` entries rather than a single aggregated station selector.

### 3.3 Direction-Level Representation

When an alert applies to a specific direction of travel at a station, the corresponding `informed_entity` entry should include `direction_id`.

When an alert applies to both directions at a station, `direction_id` may be omitted.

## 4. Field Definitions

The following fields are relevant to the Stations Affected representation.

| Field | Type | Cardinality | Definition |
| --- | --- | --- | --- |
| `stop_id` | string | required | Identifies the directly affected station. |
| `direction_id` | integer | conditional | Identifies the affected direction of travel when the impact is direction-specific. |

### 4.1 `stop_id`

`stop_id` should contain a valid GTFS stop identifier for the affected subway station.

Consumers should interpret the presence of `stop_id` as indicating that the specified station is directly affected by the alert.

### 4.2 `direction_id`

`direction_id` should use the following value mapping:

| Value | Meaning |
| --- | --- |
| `0` | Northbound |
| `1` | Southbound |

If `direction_id` is omitted, consumers should interpret the affected station as impacted in both directions.

## 5. Processing Rules

### 5.1 Producer Requirements

Producers should emit one `informed_entity` entry per directly affected station.

Producers should include `route_id` when the affected station context is route-specific.

Producers may include additional standard or extension fields on the same `informed_entity` entry, provided those fields do not contradict the station-level meaning defined in this document.

### 5.2 Consumer Requirements

Consumers should evaluate each `informed_entity` entry independently.

Consumers should not infer that all stations on a route or route segment are affected solely because one or more station-specific `informed_entity` entries are present.

Consumers should prioritize station-specific entries in rider-facing messaging when presenting impacts from Planned Work or similar disruptions.

Consumers should use `direction_id` to distinguish northbound-only, southbound-only, and bidirectional impacts.

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

## 7. Implementation Notes

This representation is initially introduced for NYCT Planned Work.

MTA may extend this representation to additional subway alert categories in future feed updates without changing the encoding model defined in this document.
