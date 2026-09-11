---
name: overassigned
description: Lists eResource Scheduler people or groups (team, location, department, role) booked or logged above working capacity. Use when someone says "overassigned", "which team is overassigned", "which location is overassigned", "which department is overassigned", "which role is overassigned", "overassigned teams", "grouped by team", "overallocated", "overbooked", "who is over capacity", or "overload".
---

# Overassigned

Read-only. Print in chat. Do not change bookings or timesheets.

Use `ers_report_get`. Do not call `ers_booking_search`, `ers_timesheet_search`, or `ers_resource_search`. Do not invent ids. Never send `organizeBy` on utilization — rejected (`WRONG_PARAMS`).

If MCP is missing or auth fails: stop. Tell the user to connect at **Settings → Tools & MCP → eRS → Connect**.

## Grain — check this first (do not default to people)

Scan the user message **before** picking people. People is only the fallback when **none** of team / location / department / role appear.

| If the message has | Grain | First column |
|---|---|---|
| team / teams | **Team** | Team |
| location / locations | **Location** | Location |
| department / departments | **Department** | Department |
| role / roles | **Role** (primary) | Role |
| none of the above | **people** | Name |

These are all Team — do **not** list people:

- "which team is overassigned"
- "what team is over capacity"
- "overassigned teams"
- "grouped by team"

Same pattern for location, department, and role. You do **not** need the words "grouped by". "Which X" is enough.

If two group nouns appear, group by the one after **which/what** or **grouped by**. A named value is a filter, not the grain ("overassigned teams in UserLoc-01" → grain Team, filter that location). If still two grains, ASK the field **names**.

Never print a people table when grain is Team / Location / Department / Role.

## Resolve the field

When grain is not people:

1. `ers_type_get` `entity=resource`. Match grain to `display_name` (not a hardcoded table): Team → `udf_team`, Location → `udf_location`, Department → `udf_department`, Role → `primary_role` (filter code `roles`). One match → that `code`. Zero or 2+ → ask **names**.
2. Keep `options[]` (id → label). Group buckets often have `id` only (`351`, not "Technical").

## Report call

- `report=utilization`
- `view=resource`
- `startDate` + `endDate` (`yyyy-MM-dd`)
- `limit=500`; page `offset` while `has_more`

**People:** no `resourceFilters`. Use `name`, `total_capacity_hrs`, `total_planned_hrs` / `total_actual_hrs`.

**Team / location / department / role:** `resourceFilters` `{ "code": "<code>", "values": [<all real option names for that field>] }` so `report.groups` and `group_values` exist. Code-only is rejected. Do not send `organizeBy`. If the user named one option, `values` is that option only.

`values` are Planned Utilization option **names** only (Technical, Operations, …). Never send `Team Undefined`, `Location Undefined`, `Role Undefined`, or `{Field} Undefined` — not an option; the tool returns `VALIDATION_ERROR` ("No udf_team option … matches \"Team Undefined\""). People with no team still come back in `groups.udf_team` with `is_undefined=true` — that bucket is output-only. Same for location / department / role. If a call failed because Undefined was in `values`, drop it and recall **once**.

| User said | `reportType` | `data` | What to print |
|---|---|---|---|
| Bookings / scheduled / planned / allocated only | `planned` | `planned,capacity` | Bookings section |
| Timesheets / actuals / logged time, **or nothing** | `planned_vs_actual` | `planned,actual,capacity` | Timesheets section, or **both** sections from this one payload |

`planned_vs_actual` already includes planned hours. Do **not** also call `planned`. Never two reportTypes. Never `report=timesheet`.

If `planned_vs_actual` fails, fall back to one `planned` call and say actuals were not available.

## Hours

**People:** Capacity = `total_capacity_hrs`. Booked = `total_planned_hrs`. Logged = `total_actual_hrs` (only on `planned_vs_actual`).

**Grouped:** name = `data.name`. Capacity = `display_units.planned.total.capacity_hrs`. Booked = `display_units.planned.total.hrs`. Logged = `display_units.actual.total.hrs`. Bucket = `group_values.<code>`.

Overbooked = load − capacity. Keep if **> 0.25h**.

**Group row:** page until complete; sum capacity and load per bucket. Label from `groups.<code>[].label` or type options. Rows with `is_undefined=true` (no team / location / department / role set) → first column `Team Undefined` (or Location / Department / Role Undefined). `%` = that group's load ÷ capacity × 100. `groups.*` has no hours — do not print `resource_ids`. Copy `data.by_role` only as a hint; it is page-sliced — still sum from resources.

Round hours to 1 decimal (drop `.0`). Round % to a whole number.

## Dates

User's range, else **current week** (Mon–Sun). Named person (`resourceName`) only on **people** grain.

## Output

Markdown table in chat. No code fence. No API field dumps. Overassigned rows only. Sort most overbooked first. Cap 25; then `+<n> more`.

Worked person: 40h capacity, 176h booked → **136h (440%)**.  
Worked team: Technical 200h capacity, 248h booked → **48h (124%)**.

```markdown
# Overassigned — <start> to <end>

Basis: <bookings | timesheets | bookings and timesheets>
Grouped by: <Team | Location | Department | Role | people>

## Bookings (planned vs capacity)

- Overassigned: <n>

| Team | Capacity | Booked | Overbooked |
|---|---:|---:|---:|
| Technical | 200h | 248h | 48h (124%) |
```

First column = grain (Team / Location / Department / Role / Name). Empty: `No overassigned <teams | locations | departments | roles | people> in <start> to <end>.`

If a tool fails: say so and print a partial table.

People: [examples/01-current-week.md](examples/01-current-week.md). Grouped: [examples/02-grouped-by.md](examples/02-grouped-by.md).
