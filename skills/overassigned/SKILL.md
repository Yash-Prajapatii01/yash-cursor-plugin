---
name: overassigned
description: Lists eResource Scheduler people, teams, or other groups booked or logged above working capacity. Use when someone says "overassigned", "overallocated", "overbooked", "who is over capacity", "overassigned teams", "grouped by location", "grouped by team", "working more than capacity", or "overload".
---

# Overassigned

Read-only list of people, teams, or other groups whose load is above capacity. Print in chat. Do not change bookings or timesheets.

Use `ers_report_get`. Do not call `ers_booking_search`, `ers_timesheet_search`, or `ers_resource_search`. Do not invent ids. Never send `organizeBy` on utilization — the tool rejects it (`WRONG_PARAMS`). Grouping is `report.groups` + `group_values` on the same utilization call.

If MCP is missing or auth fails: stop. Tell the user to connect at **Settings → Tools & MCP → eRS → Connect**.

## Grain

| User said | Grain | First column |
|---|---|---|
| nothing / who / people / resources | **people** (default) | Name |
| grouped by location / by team / by department / by role | **that field** | Location, Team, … |
| overassigned teams / groups | field named Team, else Group | Team |

`organizeBy` is **project_progress only**. For utilization, "grouped by X" means: resolve X to a resource field code, then roll up hours from `group_values.<code>`. Copy `data.by_role` only as a hint for **role**; still page resources if `has_more` (by_role on a page is not the whole tenant).

## Resolve the group field

When grain is not people:

1. `ers_type_get` `entity=resource` (list types; Personnel fields are enough). Match the user's words to `display_name` (Location → `udf_location`, Team → `udf_team`, Department → `udf_department`, Role → `primary_role` / `roles`). One match → use that `code`. Zero or 2+ → ask with the **names**, not ids.
2. Keep `fields[].options[]` (id → label). Group buckets often have only `id` (e.g. `406`) and no `label`.

## Report call

Always:

- `report=utilization`
- `view=resource` (capacity only exists here)
- `startDate` + `endDate` (`yyyy-MM-dd`)
- `limit=500`; page `offset` while `has_more`

**People (no grouped-by):** do **not** send `resourceFilters`. Rows have `name`, `total_capacity_hrs`, `total_planned_hrs` / `total_actual_hrs`.

**Grouped by:** send `resourceFilters` `{ "code": "<udf_*>", "values": [<all option names for that field>] }`. That switches to the UI utilization endpoint so `report.groups` and `group_values` exist. Code-only is rejected (needs values). Do not treat one option as "only this location" unless the user named that location. Include `{Field} Undefined` (`is_undefined=true`) — never drop it.

Omit `reportType` **only** if this skill cannot map the user's words. Then ASK `planned` vs `planned_vs_actual`. Otherwise pass it on the first call.

| User said | `reportType` | Load |
|---|---|---|
| Bookings / scheduled / planned / allocated | `planned` | planned hours |
| Timesheets / actuals / logged time | `planned_vs_actual` | actual hours |
| Nothing | **both** — two calls | planned, then actual |

Timesheet overload is utilization `planned_vs_actual`, not `report=timesheet`.

`data`: `planned,capacity` for bookings; `planned,actual,capacity` for timesheets.

## Hours (do not dump API fields)

**People, unscoped:** Capacity = `total_capacity_hrs`. Load = `total_planned_hrs` or `total_actual_hrs`.

**Grouped / UI payload:** name = `data.name`. Capacity = `display_units.planned.total.capacity_hrs`. Booked = `display_units.planned.total.hrs`. Logged = `display_units.actual.total.hrs`.

Per person: Overbooked = load − capacity. Keep if **> 0.25h**.

**Group row:** sum capacity and load for every resource with that `group_values.<code>` (page until complete). Label from `groups.<code>[].label` or type option name; Undefined → `Location Undefined` / `Team Undefined`. Overbooked = group load − group capacity. Keep if **> 0.25h**. `%` = group load ÷ group capacity × 100 (that group only).

Round hours to 1 decimal (drop `.0`). Round % to a whole number.

Never rebuild from bookings. `groups.*` has membership (`resource_ids`) only — not hours.

## Dates

User's range, else **current week** (Mon–Sun). Named person: `resourceName` (people grain). Named location/team: `resourceFilters` values = that one option.

## Output

Markdown table in chat. No code fence. No `{name, total_capacity_hrs, ...}`. Overassigned rows only. Sort most overbooked first. Cap 25; then `+<n> more`.

Worked person: Resource 18, 40h capacity, 176h booked → **136h (440%)**.

```markdown
# Overassigned — <start> to <end>

Basis: <bookings | timesheets | bookings and timesheets>
Grouped by: <people | Location | Team | …>

## Bookings (planned vs capacity)

- Overassigned: <n>

| <Name or Location or Team> | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| UserLoc-01 | 400h | 520h | 120h (130%) |

## Timesheets (actual vs capacity)

- Overassigned: <n>

| <Name or Location or Team> | Capacity | Logged | Overbooked |
|---|---:|---:|---:|
| Technical | 200h | 248h | 48h (124%) |
```

Empty: `No overassigned <people | locations | teams> in <start> to <end>.`

If a tool fails: say so and print a partial table.

People shape: [examples/01-current-week.md](examples/01-current-week.md). Grouped by: [examples/02-grouped-by-location.md](examples/02-grouped-by-location.md).
