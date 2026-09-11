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

From each `resources[]` row, compute — do not print the API fields:

| API field | Table column |
|---|---|
| `name` | Name |
| `total_capacity_hrs` | Capacity |
| `total_planned_hrs` | Booked (bookings section) |
| `total_actual_hrs` | Logged (timesheets section) |
| planned − capacity | Overbooked hours |
| planned ÷ capacity × 100 | Overbooked % |

Resource 18: 176 booked / 40 capacity → **136h (440%)**. Chris Rose 40/40 → drop the row.

**Wrong (never do this):**

```
{name, total_capacity_hrs, total_planned_hrs}
Resource 18, 40, 176
```

**Right** — markdown table in chat, not inside a code fence:

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings and timesheets

## Bookings (planned vs capacity)

- Overassigned: 1

| Name | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| Resource 18 | 40h | 176h | 136h (440%) |

## Timesheets (actual vs capacity)

No overassigned resources in 2026-09-07 to 2026-09-13.

If nobody is over on either side:

# Overassigned — 2026-09-07 to 2026-09-13

Basis: bookings and timesheets

No overassigned resources in 2026-09-07 to 2026-09-13.

## What this demonstrates

- One `ers_report_get` per method; no "list all reports" first call.
- Bookings → `planned`. Timesheets → `planned_vs_actual` (not `report=timesheet`).
- Default "nothing named" → both sections.
- User sees Name / Capacity / Booked or Logged / Overbooked — never raw field tuples.
- Empty state is one line per section, not invented rows.
- People only here. Team / location / department / role: [02-grouped-by.md](02-grouped-by.md).
