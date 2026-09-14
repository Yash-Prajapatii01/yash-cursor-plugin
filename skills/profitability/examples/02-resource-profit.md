# Example — which resource is profitable (bookings and timesheets)

**User:** which resource is profitable

Grain = **People**. Profit — if `needs_cost_rate_source`, ASK 0/1/2/3, then recall. One `planned_vs_actual` (plan_financial + actual_financial on the same payload).

```
ers_report_get
  report=financial
  reportType=planned_vs_actual
  startDate=2026-09-07
  endDate=2026-09-13
  costRateSource=0
```

(`costRateSource` only after Admin omit-first, or after the user picked.)

Per resource, sum `dailyUtilCost` (bookings) and `dailyActualUtilCost` (timesheets). Each line `[work_cost, revenue, record_id, project_id, …]`. Map `project_id` → `report.projects[].title`. `%` = profit ÷ revenue when revenue > 0 (`admin.profit_calculation`).

Top 10 names: `ers_rate` with the resource **name** on `ownerId`. Copy the card. Never hours × rate for profit.

**Wrong:** `reportType=planned` and `actual` as two calls when they asked for both.  
**Wrong:** `view=resource` on financial.

**Right:**

# Profit — 2026-09-07 to 2026-09-13

Basis: bookings and timesheets
Currency: USD · Profit %: Profit / Revenue
Cost rate: Resource

| Name | Rate | Projects | Profit | Profit % |
|---|---|---|---:|---:|
| Chris Rose | 70 USD/h | Employee Management System | -1750 | — |

(`—` when revenue is 0 — do not divide.)

Bookings-only → `reportType=planned` (plan_financial). Timesheets-only → `actual` (actual_financial).
