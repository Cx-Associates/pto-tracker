# CxA PTO Tracker

A lightweight PTO tracking tool for the CxA team. Employees can view their balance, log time off, and schedule future PTO. Admins can manage staff records, add entries, and export reports.

**Live app:** https://cx-associates.github.io/pto-tracker

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

## Architecture

See [`CxA_PTO_Tracker_Architecture.md`](CxA_PTO_Tracker_Architecture.md) for full technical documentation.
