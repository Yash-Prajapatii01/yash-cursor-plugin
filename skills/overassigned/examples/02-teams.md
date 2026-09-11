# Example — overassigned teams this week

**User:** which teams are overassigned

**Agent:** resolve Team from the tenant form, then one utilization call **per team option**. Do not page people. Do not send `organizeBy`.

```
ers_type_get
  entity=resource
```

Human type (Personnel) → field display_name Team, `code=udf_team`, `id=66`, options include Technical `351`. Those ids come from this tenant — never reuse them on another account.

Wrong filter (returns the whole tenant, mixed teams):

```
resourceFilters: { "code": "udf_team", "values": ["Technical"] }
```

Right filter:

```
ers_report_get
  report=utilization
  reportType=planned
  view=resource
  data=planned,capacity
  startDate=2026-09-07
  endDate=2026-09-13
  limit=1
  resourceFilters: { "code": "udf_team", "id": 66, "filterType": 72, "multiValue": [351] }
```

Repeat for each Team option. Same dates with `reportType=planned_vs_actual` and `data=planned,actual,capacity` if the user also wants timesheets (or said nothing).

Read totals, not `resources[]`:

| Total | Table |
|---|---|
| `total_count` | People |
| `totalCapacity_display_units.total.hrs` | Capacity |
| `totalUtilization_display_units.total.hrs` | Booked |
| `totalActualUtilization_display_units.total.hrs` | Logged |
| booked − capacity | Overbooked |

Worked tenant row: Technical 421 people, 3320h booked / 16840h capacity → **not** overassigned (omit from the table).

**Right** — markdown table in chat, not inside a code fence:

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings
Grain: teams

## Bookings (planned vs capacity)

- Overassigned: 1

| Team | People | Capacity | Booked | Overbooked |
|---|---:|---:|---:|---:|
| Sales & Marketing | 80 | 3200h | 4100h | 900h (128%) |

If none:

No overassigned teams in 2026-09-07 to 2026-09-13.

## What this demonstrates

- Teams are utilization + `resourceFilters`, not `view=team`.
- Field code/id/option ids from `ers_type_get`, not hardcoded.
- Totals are the whole team even when `limit` is 1.
- Under-capacity teams are omitted unless the user named that team.
