---
name: profitability
description: Ranks eResource Scheduler projects, teams, or people by revenue or profit. Use when someone says "earning more revenue", "which project is making money", "which team is earning", "which resource is profitable", "profitability", "top revenue", "margin", or "profit %".
---

# Profitability

Read-only. Print in chat. Do not change bookings, timesheets, or rates.

Use `ers_report_get` `report=financial`. Never `ers_booking_search`, `ers_timesheet_search`, `ers_resource_search`. Never `hours × rate` (if `FINANCIAL_ACCESS_DENIED`, stop). Never `organizeBy` or `view` on financial — both are `WRONG_PARAMS`.

If MCP is missing or auth fails: stop. Tell the user to connect at **Settings → Tools & MCP → eRS → Connect**.

## Grain

| User said | Grain | Data |
|---|---|---|
| project / projects | **Project** | Roll up allocation lines by project id |
| team / teams / groups | **Team** (field named Group if no Team) | COPY `report.groups.udf_team` (money is already on the bucket) |
| department / location / role | that field | COPY `report.groups.<code>` |
| resource / person / who is profitable | **People** | Sum each `resources[]` row |
| earning / revenue and no grain | **Project** | |
| profitable and no grain | **People** | |

Do not list people when grain is Team / Project. `project_groups` has **no** money — do not use it to rank revenue.

## Report call (plan_financial vs actual_financial)

`report=financial`. Dates: named range, or named month → first–last day, else **current week** (Mon–Sun).

| User said | `reportType` | Meaning |
|---|---|---|
| Bookings / scheduled / planned / allocated | `planned` | plan_financial |
| Timesheets / actuals / logged | `actual` | actual_financial |
| Both / nothing | `planned_vs_actual` | both on one payload — do not also call `planned` or `actual` |

Never omit `reportType` (that only returns `needs_calculation_method`). Never `report=timesheet` or `report=utilization` for money.

**Cost rate:**

- **Revenue only** (“earning”, “top revenue”, “which project/team gives more revenue”): send `costRateSource=0` on the **first** call. Revenue does not use cost rate — `0` only unblocks this tenant when Admin has none set. Do **not** omit, do **not** make a second call, do **not** ASK.
- **Profit / profitable / % / margin:** omit `costRateSource` on the first call. If `needs_cost_rate_source`, ASK which `available_cost_rate_sources` (0 Resource, 1 Role, 2 Resource then Role, 3 Role then Resource). Do not assume Resource. Then recall.
- Copy `admin.currency` and `admin.profit_calculation` (e.g. Profit / Revenue). Never invent a formula.

**Never print to the user:** `needs_cost_rate_source`, `ask_user`, “Admin doesn’t have a cost-rate source”, “Recalling with Resource”, or that a cost rate was missing. That is an internal recall, not an error.

## Hours vs money

COPY group money. Do not rebuild from bookings.

| `reportType` | Work cost | Revenue | Profit |
|---|---|---|---|
| `planned` | `cost` | `revenue` | `profit_loss` |
| `actual` / both | `actual_cost` (also copied as `cost`) | `actual_revenue` | `actual_profit_loss` |

Work cost **excludes Bench**. `total_cost` / `actual_total_cost` = work + bench. Name the column (Planned Cost vs Actual Cost). Bare “cost”: ASK Actual vs Bench before ranking.

`%` = profit ÷ revenue × 100 when revenue **> 0**, matching `admin.profit_calculation`. If revenue is 0, show profit as money and `%` as `—` (do not divide).

Keep zero-money buckets (including `{Field} Undefined`). Never put `Team Undefined` in a filter `values` list.

## Team / department / location / role

COPY `report.groups.<code>` (`udf_team`, `udf_department`, `udf_location`, `primary_role`). Fields: `id`, `label` (may be missing), `revenue` / `actual_revenue`, `profit_loss` / `actual_profit_loss`, `cost` / `actual_cost`.

Label: `label` if present, else option name from `ers_type_get` **one** type id (not the full catalog). `is_undefined` → `Team Undefined` in the **table only**.

## Project

`report.projects[]` has titles, not money. `project_groups` is membership only — do not rank from it.

Sum allocation lines on each `resources[]` row:

- Planned: `dailyUtilCost`
- Actual: `dailyActualUtilCost`
- Both: both arrays on the same row (`planned_vs_actual`)

Each day is `[day_work_cost, [lines…], day_revenue]`. Each line is `[work_cost, revenue, record_id, project_id, …]` (length 6). **Index 3 is project id.** Never treat `record_id` (index 2) as a project. Map `project_id` → `title` from `report.projects`. Check `sum(planned revenue) == sum(report.totalRevenue)` and actual vs `totalActualRevenue`.

**Never add planned + actual.** Rank by planned revenue unless the user said timesheets / actuals / logged. On `planned_vs_actual`, always show both columns. If the actual leader is a different project, name both.

Revenue ranking: print rows with planned or actual **> 0**. Mention how many projects were 0. Do not list the full catalog.

## People (profitable resources)

For each `resources[]`: name = `data.name`. Sum the same daily lines for work cost + revenue. Profit and `%` as above.

**Projects they work:** unique `project_id` titles from those lines.

**Rates:** do not compute profit from rates. When the user asked for rates (or this people output), `ers_rate` with the **resource name** on `ownerId` for the top rows only (cap 10). Copy the card; never `hours × rate`.

## Chart + list

Markdown in chat. Then a ranked table. Do not wrap the table in a fence.

**Never emit `pie`.** Cursor Chat does not render Mermaid pie (it falls back to flowchart and shows `Mermaid Syntax Error`). Use **`xychart-beta` bars** — that type is supported. Put the date range in the markdown heading, not in the chart title.

Emit **exactly** this shape inside a `mermaid` fence (do not copy this example as `pie`):

    xychart-beta
        title Planned revenue USD
        x-axis [ConnectSphere, DocMgmt, Goibibo, Other]
        y-axis "USD" 0 --> 7000
        bar [6000, 3750, 2000, 11210]

Rules:

- Keyword is `xychart-beta` (not `pie`, not `xychart`)
- Title: short words only (no `()`, no `—`, no dates)
- **`x-axis` labels: one word each, no spaces, no quotes.** Cursor's renderer prints quotes literally and does not rotate or wrap labels, so quoted multi-word names overlap into a smear. Abbreviate: `Document Management System` → `DocMgmt`, `Nova Social Media Revamp` → `Nova`. Full names go in the table, never in the chart
- **At most 6 bars** (5 named + `Other`). More than that collides even with short labels
- Plain `xychart-beta` only — do **not** add `horizontal` to fit longer names
- `bar` values: numbers only (`6000` not `6000 USD` or `6,000`). Negatives are OK
- `y-axis "USD" 0 --> <max>` where max is a round number ≥ the largest bar
- Cap the table at 15; then `+<n> more`
- Revenue question → bars of revenue. Profitable question → bars of profit
- If every value is 0, or two or fewer rows are non-zero, skip the chart and keep the table

```markdown
# <Revenue | Profit> — <start> to <end>

Basis: <bookings (planned) | timesheets (actual) | bookings and timesheets>
Currency: <admin.currency> · Profit %: <admin.profit_calculation>

Top planned: <name> — <amount> <currency>
Top actual: <name> — <amount> <currency>   (omit this line if actual is all 0)
```

Revenue-only table (no Profit columns — cost rate was not chosen for ranking):

| Grain | Planned | Actual |
|---|---:|---:|
| ConnectSphere | 6000 | 0 |

People + profit table: **Name | Rate | Projects | Profit | Profit %**.

Empty / all revenue 0: skip the chart; `No revenue in <start> to <end>.` Still list profit/cost if those are non-zero.

If a tool fails: quote the error; do not retry the same `organizeBy`/`view` payload.

Examples: [examples/01-revenue-by-team.md](examples/01-revenue-by-team.md), [examples/02-resource-profit.md](examples/02-resource-profit.md), [examples/03-revenue-by-project.md](examples/03-revenue-by-project.md).
