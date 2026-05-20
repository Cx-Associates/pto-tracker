# CxA PTO Tracker

A lightweight PTO tracking tool for the CxA team. Employees can view their balance, log time off, and schedule future PTO. Admins can manage staff records, add entries, and export reports.

**Live app:** https://cx-associates.github.io/pto-tracker

---

## Features

**For employees:**
- View current PTO balance and estimated year-end balance
- See accrued, rollover, taken, and scheduled hours at a glance
- Log PTO taken (past dates only) or schedule upcoming time off
- Enter date ranges — weekday hours calculated automatically
- Project your balance to any future date
- View full activity history with past and upcoming entries clearly separated
- Change your own PIN

**For admins:**
- Dashboard overview of all staff balances, accrual rates, and estimated rollovers
- Add, edit, and manage employee records and PTO entries
- Generate and export reports to CSV
- View a calendar showing all staff PTO across the team
- Reset employee PINs
- Change the admin PIN

---

## How it works

The app is a single HTML file with no build step. It runs entirely in the browser and stores all data in Supabase. Hosting is provided by GitHub Pages.

---

## Updating the app

1. Go to the repo on GitHub and click `index.html`
2. Click the pencil (Edit) icon in the top right
3. Select all the code and replace it with the new version
4. Click **Commit changes**, then **Commit changes** again in the dialog
5. The live site updates within ~60 seconds

---

## Configuration

| Setting | Value |
|---|---|
| Supabase project | `ykddpajaqftrcmmhizoq.supabase.co` |
| Supabase table | `pto_data` |
| Default admin PIN | `1234` — change this on first login via Admin → Settings |
| PTO year | January 2 – January 1 |
| Rollover cap | 40 hours |

---

## Accrual policy

| Tenure | Full-time annual allotment |
|---|---|
| 0–4 years | 240 hrs/yr |
| 4–8 years | 256 hrs/yr |
| 8+ years | 280 hrs/yr |

Part-time employees accrue proportionally based on their weekly hours. Individual overrides can be set per employee in the Admin → Staff panel.

---

## Adding a new employee

In the live app, go to **Admin → Staff → Add employee**. Enter their full name, then use the Edit button to set their hire date, weekly hours, rollover balance, and PIN. No code changes required.

---

## Maintenance note

Supabase is enforcing explicit GRANT statements on all existing projects from **October 30, 2026**. Before that date, run the following in the Supabase SQL Editor:

```sql
grant select, insert, update
  on public.pto_data
  to anon;
```

---

## Architecture

See [`CxA_PTO_Tracker_Architecture.md`](CxA_PTO_Tracker_Architecture.md) for full technical documentation.
