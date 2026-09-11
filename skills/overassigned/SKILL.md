---
name: overassigned
description: Lists eResource Scheduler resources booked or logged above their working capacity. Use when someone says "overassigned", "overallocated", "overbooked", "who is over capacity", "working more than capacity", or "overload".
---

# Overassigned

Read-only list of people (and other resources) whose load is above capacity. Print in chat. Do not change bookings or timesheets.

Use only `ers_report_get`. Do not call `ers_booking_search` or `ers_timesheet_search` for this job. Do not invent resource ids.

If MCP is missing or auth fails: stop. Tell the user to connect at **Settings → Tools & MCP → eRS → Connect**.

## Correct flow (do not invent a "list all reports" step)

`ers_report_get` runs **one** report per call. Overassigned is always:

- `report=utilization`
- `view=resource` (capacity exists only on the resource view)
- `startDate` + `endDate` (`yyyy-MM-dd`)

Omit `reportType` **only** if this skill cannot map the user's words. Then the tool returns `needs_calculation_method` — ask them to pick `planned` or `planned_vs_actual`. This skill normally maps the method itself (below), so pass `reportType` on the first call.

## Which reportType

| User said | `reportType` | Load column | Why |
|---|---|---|---|
| Bookings / scheduled / planned / allocated | `planned` | `total_planned_hrs` | Booked hours vs capacity |
| Timesheets / actuals / logged time | `planned_vs_actual` | `total_actual_hrs` | Logged hours vs capacity |
| Nothing (plain "overassigned") | **both** — two calls | planned, then actual | Show two sections |

Timesheet overload is **utilization** `planned_vs_actual`, not `report=timesheet`.

`data`: `planned,capacity` for bookings; `planned,actual,capacity` for timesheets.

Copy hours from `data.report.resources[]`. Never rebuild utilization from bookings. Overbooked hours = load − `total_capacity_hrs`. Treat under **0.25h** as not overassigned (float noise).

If `has_more`, page with `offset`/`limit` until complete. Cap the printed table at 25; note the remainder.

## Dates

Use the range the user named. Else **current week** (Mon–Sun). Optional named person: `resourceName`.

## Output

Overassigned only (load > capacity). Sort most overbooked hours first.

```markdown
# Overassigned — <range>

Basis: <bookings | timesheets | bookings and timesheets>

## Bookings (planned vs capacity)

- Overassigned: <n>

| Name | Capacity | Overbooked |
|---|---:|---:|
| <name> | <total_capacity_hrs>h | <planned − capacity>h (<pct>%) |

## Timesheets (actual vs capacity)

- Overassigned: <n>

| Name | Capacity | Overbooked |
|---|---:|---:|
| <name> | <total_capacity_hrs>h | <actual − capacity>h (<pct>%) |
```

Omit the section you did not run. `pct` = load ÷ capacity × 100 from **that resource's** hours (not hours ÷ sum of all hours).

Empty: `No overassigned resources in <range>.`

If a needed tool fails: say so and print a partial list from what you got.

If the output shape is unclear, read [examples/01-current-week.md](examples/01-current-week.md).
