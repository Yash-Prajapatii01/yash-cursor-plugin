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

| If the message has | Grain | First column | First `code` to try |
|---|---|---|---|
| team / teams | **Team** | Team | `udf_team` (or `Team`) |
| location / locations | **Location** | Location | `udf_location` (or `Location`) |
| department / departments | **Department** | Department | `udf_department` (or `Department`) |
| role / roles | **Role** (primary) | Role | `roles` |
| none of the above | **people** | Name | — |

"which team is overassigned" / "overassigned teams" / "grouped by team" → Team, **not** people. Same for location, department, role. You do not need the words "grouped by".

If two group nouns appear, group by the one after **which/what** or **grouped by**. A named value is a filter ("teams in UserLoc-01" → grain Team, filter that location). If still two grains, ASK the field **names**.

Never print a people table when grain is Team / Location / Department / Role.

## Do not start with `ers_type_get`

Do **not** call `ers_type_get entity=resource` with no `id` (full type catalog, large). Utilization screen-data already knows Team / Location / Department / Role.

1. Call `ers_report_get` with `resourceFilters.code` from the table above.
2. `values`: only **real option names**. If the user named one (Technical), send that one. If they asked "which team" (all teams), omit `values` on the first try. **Never** put `Team Undefined`, `Location Undefined`, or `{Field} Undefined` in `values` — not an option; the call fails.
3. If the tool returns `VALIDATION_ERROR` / `fieldErrors` with a valid-code or valid-values list: recall **once** using that list. Still no Undefined in `values`.
4. Need labels for numeric ids (`351`) and `groups[].label` is missing: then `ers_type_get` **one** type `id` (Personnel), not the catalog.

Undefined is **output only**: after a successful report, still show `groups.*` rows with `is_undefined=true` as `Team Undefined` (or Location / Department / Role Undefined).

## Report call

- `report=utilization`
- `view=resource`
- `startDate` + `endDate` (`yyyy-MM-dd`)
- `limit=500`; page `offset` only if you still need hours (below)

**People:** no `resourceFilters`. Use `name`, `total_capacity_hrs`, `total_planned_hrs` / `total_actual_hrs`.

**Grouped:** `resourceFilters` as above so `report.groups` and `group_values` exist. Do not send `organizeBy`.

| User said | `reportType` | Load |
|---|---|---|
| Bookings / scheduled / planned / allocated | `planned` | planned hours |
| Timesheets / actuals / logged time | `planned_vs_actual` | actual hours |
| Nothing | **both** | planned, then actual |

Timesheet overload is utilization `planned_vs_actual`, not `report=timesheet`. `data`: `planned,capacity` or `planned,actual,capacity`.

## Hours

**People:** Capacity = `total_capacity_hrs`. Load = `total_planned_hrs` or `total_actual_hrs`.

**Grouped — copy totals if they exist, do not invent them.** Check `groups.<code>[]` (and `data.by_role` for Role) for `capacity_hrs` plus `planned_hrs` / `actual_hrs` / `hrs`. If those fields are present **and** `has_more` is false (or the totals are clearly tenant-wide, not one page): copy them. Stop. Do not page people.

Today `groups.udf_team` is usually membership only (`id`, `resource_count`, `resource_ids`) — **no hours**. `by_role` hours are often **this page**, not the tenant. Then you must sum:

- Capacity = `display_units.planned.total.capacity_hrs`
- Booked = `display_units.planned.total.hrs`
- Logged = `display_units.actual.total.hrs`
- Bucket = `group_values.<code>`
- Label = `groups.<code>[].label` or option name; else `Team Undefined` when `is_undefined`

Page `limit=500` until `has_more` is false. If you stop early, say the table is **partial**. Do not print `resource_ids`.

Overbooked = load − capacity. Keep if **> 0.25h**. `%` = that row’s load ÷ capacity × 100. Round hours to 1 decimal (drop `.0`). Round % to a whole number.

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

First column = grain. Empty: `No overassigned <teams | locations | departments | roles | people> in <start> to <end>.`

If a tool fails: quote the error; if it lists valid `values`, recall without Undefined. Do not retry the same bad payload.

People: [examples/01-current-week.md](examples/01-current-week.md). Grouped: [examples/02-grouped-by.md](examples/02-grouped-by.md).
