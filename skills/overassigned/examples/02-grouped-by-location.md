# Example — utilization grouped by location (then overassigned)

**User:** show me the utilization grouped by location  
(same path for "overassigned teams" / "grouped by team" — swap Location for Team / `udf_team`)

Do **not** send `organizeBy` on utilization. That returns `WRONG_PARAMS`.

**Agent:**

1. `ers_type_get` `entity=resource` → display name Location = `udf_location` (options include `406` UserLoc-01). Team = `udf_team`.
2. `ers_report_get` — UI filters unlock `report.groups` (unscoped POST `/v1/utilization` only has `groups.primary_role`).

```
ers_report_get
  report=utilization
  reportType=planned
  view=resource
  data=planned,capacity
  startDate=2026-09-07
  endDate=2026-09-13
  limit=500
  resourceFilters={"code":"udf_location","values":["UserLoc-01","UserLoc-02", … all Location option names]}
```

3. If `has_more`, raise `offset` and repeat until complete. Sum hours yourself.

From each `resources[]` row (UI payload):

| Read | Use as |
|---|---|
| `data.name` | person name (if listing people) |
| `display_units.planned.total.capacity_hrs` | capacity |
| `display_units.planned.total.hrs` | booked |
| `group_values.udf_location` | bucket id (`406`, or `"Undefined"`) |
| `report.groups.udf_location[]` | membership; `label` may be missing — map `406` → UserLoc-01 from type options |
| grouping_note | keep `Location Undefined` |

`groups.udf_location[]` has **no hours**. Roll up:

UserLoc-01 (id 406): sum capacity 400h, sum booked 520h → Overbooked **120h (130%)**.  
A location at 400h booked / 400h capacity → drop.

**Wrong:** `organizeBy=location` or `organizeBy=udf_location`.  
**Wrong:** dump `{id, resource_count, resource_ids}`.  
**Wrong:** one page only (`has_more` true).

**Right** — markdown table in chat, not in a code fence:

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings
Grouped by: Location

## Bookings (planned vs capacity)

- Overassigned: 1

| Location | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| UserLoc-01 | 400h | 520h | 120h (130%) |

## What this demonstrates

- Grouped-by on utilization = `resourceFilters` + `report.groups` / `group_values`, not `organizeBy`.
- Teams: same call with `code=udf_team` and Team option names.
- Hours come from `display_units`, labels from type options when groups omit `label`.
