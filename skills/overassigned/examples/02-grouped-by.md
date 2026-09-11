# Example — which team is overassigned

**User:** which team is overassigned

This is **Team** grain. Do not list people. Do not send `organizeBy`. Do not call `ers_type_get` first.

**Agent:**

1. Grain = Team. Skip people.
2. First report — real option names only, **no** `Team Undefined` in `values`:

```
ers_report_get
  report=utilization
  reportType=planned
  view=resource
  data=planned,capacity
  startDate=2026-09-07
  endDate=2026-09-13
  limit=500
  resourceFilters={"code":"udf_team"}
```

3. If that errors because `values` are required, recall with the **valid option names from the error** (Technical, Operations, …). Never add `Team Undefined`.
4. Hours: if `groups.udf_team[]` has `capacity_hrs` and `planned_hrs`, copy and stop. If it only has `id` / `resource_count` / `resource_ids`, sum `display_units.planned.total` per `group_values.udf_team` and page while `has_more`. Show `is_undefined` rows as **Team Undefined** in the table (output only).

**Wrong:** `ers_type_get entity=resource` with no id before the report.  
**Wrong:** `values` including `"Team Undefined"`.  
**Wrong:** a people Name / Capacity / Booked table.  
**Wrong:** `organizeBy=udf_team`.

**Right:**

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings
Grouped by: Team

## Bookings (planned vs capacity)

- Overassigned: 1

| Team | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| Technical | 200h | 248h | 48h (124%) |

Location / department / role: same steps; first `code` is `udf_location` / `udf_department` / `roles`.

## What this demonstrates

- "which team is overassigned" → Team, not people.
- Undefined is printed from `groups`, never sent as a filter value.
- No type catalog up front. Copy group hour fields when they exist; otherwise sum `display_units`.
