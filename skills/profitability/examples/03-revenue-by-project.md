# Example — which project gives more revenue

**User:** which project give more revenue

Grain = **Project**. Revenue-only. Current week. Do not send `organizeBy` or `view`. Send `costRateSource=0` on the **first** call. Never ASK. Never print `needs_cost_rate_source` / “Admin doesn’t have a cost-rate source” / “Recalling with Resource”.

```
ers_report_get
  report=financial
  reportType=planned_vs_actual
  startDate=2026-09-14
  endDate=2026-09-20
  costRateSource=0
```

`projects[]` = titles. Sum `dailyUtilCost` / `dailyActualUtilCost` lines: `[work_cost, revenue, record_id, project_id, …]`. Rank by **planned** revenue. Show Actual column. Never add planned + actual. Never `pie`.

**Wrong:** rank `project_groups`. **Wrong:** treat `record_id` as project. **Wrong:** one Revenue column that sums planned + actual. **Wrong:** list every 0-revenue project.

**Right:**

    xychart-beta
        title Planned revenue USD
        x-axis ["ConnectSphere", "Document Management System", "Other"]
        y-axis "USD" 0 --> 7000
        bar [6000, 3750, 13210]

# Revenue — 2026-09-14 to 2026-09-20

Basis: bookings and timesheets
Currency: USD · Profit %: Profit / Revenue

Top planned: ConnectSphere — 6000 USD of 22960 USD
Top actual: Document Management System — 4500 USD (only project with timesheet revenue)

| Project | Planned | Actual |
|---|---:|---:|
| ConnectSphere | 6000 | 0 |
| Document Management System | 3750 | 4500 |
