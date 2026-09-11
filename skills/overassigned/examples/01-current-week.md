# Example — overassigned this week (bookings + timesheets)

User said only "who is overassigned" — run **both** methods. Range defaulted to current week.

**User:** who is overassigned

**Agent:** `ers_report_get` twice — do not omit `reportType` (that only returns `needs_calculation_method`).

```
ers_report_get
  report=utilization
  reportType=planned
  view=resource
  data=planned,capacity
  startDate=2026-09-07
  endDate=2026-09-13
```

```
ers_report_get
  report=utilization
  reportType=planned_vs_actual
  view=resource
  data=planned,actual,capacity
  startDate=2026-09-07
  endDate=2026-09-13
```

Copy `data.report.resources[]`: `name`, `total_capacity_hrs`, `total_planned_hrs`, `total_actual_hrs`.

Overbooked (bookings) = planned − capacity when > 0.25h.  
Overbooked (timesheets) = actual − capacity when > 0.25h.

Sample row math: Chris Rose 40h planned / 40h capacity → **not** overassigned. Only print rows above capacity.

If none:

```markdown
# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings and timesheets

No overassigned resources in 2026-09-07 to 2026-09-13.
```

If someone is over on bookings:

```markdown
# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings and timesheets

## Bookings (planned vs capacity)

- Overassigned: 1

| Name | Capacity | Overbooked |
|---|---:|---:|
| Jane Doe | 40h | 8h (120%) |

## Timesheets (actual vs capacity)

No overassigned resources in 2026-09-07 to 2026-09-13.
```

## What this demonstrates

- One `ers_report_get` per method; no "list all reports" first call.
- Bookings → `planned`. Timesheets → `planned_vs_actual` (not `report=timesheet`).
- Default "nothing named" → both sections.
- Empty state is one line per section, not invented rows.
