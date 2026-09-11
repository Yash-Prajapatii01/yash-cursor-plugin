# Example — which team is overassigned

**User:** which team is overassigned

This is **Team** grain. Do not list people. Do not send `organizeBy`.

Same path for location / department / role — only the field code and first-column name change.

**Agent:**

1. Grain = Team (`team` in the message). Skip people.
2. `ers_type_get` `entity=resource` → display name Team = `udf_team` (e.g. Technical `351`, Administrative `356`).
3. `ers_report_get` with `resourceFilters` so `group_values.udf_team` exists:

```
ers_report_get
  report=utilization
  reportType=planned
  view=resource
  data=planned,capacity
  startDate=2026-09-07
  endDate=2026-09-13
  limit=500
  resourceFilters={"code":"udf_team","values":["Administrative","Customer Success","Operations","Research & Development","Sales & Marketing","Technical"]}
```

Never put `"Team Undefined"` in `values` (not an option → `VALIDATION_ERROR`). People with no team still appear in `groups.udf_team` as `is_undefined=true`.

4. Page while `has_more`. Sum `display_units.planned.total` per `group_values.udf_team`. Map `351` → Technical from type options. Print the no-team bucket as **Team Undefined** in the table only.

**Wrong:** a Name / Capacity / Booked people table.  
**Wrong:** `organizeBy=udf_team`.  
**Wrong:** dump `{id, resource_count, resource_ids}`.  
**Wrong:** `values` including `"Team Undefined"`.

**Right:**

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings
Grouped by: Team

## Bookings (planned vs capacity)

- Overassigned: 1

| Team | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| Technical | 200h | 248h | 48h (124%) |

Location / department / role: same steps. Resolve `display_name` → code (`udf_location`, `udf_department`, `roles` / `primary_role`), put **all** option names in `values`, first column = that field’s name.

## What this demonstrates

- "which team is overassigned" → Team, not people.
- Any of team, location, department, role uses this grouped path.
- Hours from `display_units`; labels from type options.
