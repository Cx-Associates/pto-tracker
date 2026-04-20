# CxA PTO Tracker — Architecture Documentation

## Overview

The CxA PTO Tracker is a single-file web application (`index.html`) built with React and backed by Supabase for persistent storage. It has no build step, no bundler, and no server-side code. The entire application — UI, business logic, and data access — lives in one HTML file that runs directly in any modern browser.

---

## Technology Stack

| Layer | Technology | Notes |
|---|---|---|
| UI framework | React 18 (UMD build) | Loaded from cdnjs CDN |
| Rendering | `React.createElement` (no JSX) | No Babel or compilation required |
| Database | Supabase (PostgreSQL via REST API) | Single row, single table |
| Hosting | GitHub Pages | Static file, no build pipeline |
| Styling | Inline styles + global CSS | No CSS framework |

---

## File Structure

There is one file: `index.html`. It contains three logical sections:

1. **`<style>` block** — Global CSS for buttons, inputs, selects, and sortable table headers. All component-level styles are applied inline via React's `style` prop.

2. **`<div id="root">`** — The React mount point.

3. **`<script>` block** — All application code: configuration, helpers, components, and the render call.

---

## Data Layer

### Supabase Connection

The app communicates with Supabase via two async functions:

```
dbGet()  →  fetches the single "main" row from the pto_data table
dbSet()  →  upserts that row (creates it on first run, updates it on every save)
```

The entire application state — all staff records, all PTO entries, and the admin PIN — is stored as a single JSON blob in the `data` column of one row (`id = "main"`). There is no relational structure within Supabase itself; all relationships are handled in JavaScript.

### Row Level Security

RLS is enabled on the `pto_data` table with three policies:

- `SELECT` allowed where `id = 'main'`
- `INSERT` allowed where `id = 'main'`
- `UPDATE` allowed where `id = 'main'`

No `DELETE` policy exists, preventing accidental or malicious deletion of the row.

### Data Shape

The JSON blob stored in Supabase has this shape:

```json
{
  "adminPin": "<hashed string>",
  "staff": [
    {
      "name": "Jane Smith",
      "pin": "<hashed string or null>",
      "hireDate": "2020-03-15",
      "weeklyHours": 40,
      "ptoOverride": null,
      "rolloverHours": 12.5,
      "entries": [
        {
          "id": 1713000000000,
          "type": "taken | scheduled | adjustment",
          "date": "2026-04-10",
          "endDate": "2026-04-14",
          "hours": 32,
          "note": "Spring vacation",
          "isRange": true,
          "addedBy": "admin"
        }
      ]
    }
  ]
}
```

---

## Business Logic

All PTO calculations are pure functions defined at the top of the script block. They take employee data as input and return computed values — they do not mutate state.

### Accrual Policy

```
Tenure 0–4 years:   240 hrs/yr (full-time equivalent)
Tenure 4–8 years:   256 hrs/yr (full-time equivalent)
Tenure 8+ years:    280 hrs/yr (full-time equivalent)
```

Part-time employees accrue proportionally. A 30hr/week employee at the 8+ year tier accrues `280 × (30/40) = 210 hrs/yr`.

The `ptoOverride` field allows an admin to set a custom annual allotment for any employee. When set, the override is treated as the already-prorated annual figure — the part-time scaling is not applied a second time.

### Key Calculation Functions

| Function | Purpose |
|---|---|
| `getAccrualRate(hireDate, ptoOverride)` | Returns the full-time annual hours for an employee based on tenure, or the override if set |
| `annualAllotmentDisplay(hireDate, weeklyHours, ptoOverride)` | Returns the effective annual allotment for display, prorated for part-time employees |
| `weeklyAccrual(hireDate, weeklyHours, ptoOverride)` | Returns hours accrued per week; does not double-scale overrides |
| `calcAccruedInYear(...)` | Hours accrued from Jan 2 through today (or end of year, whichever is earlier) |
| `calcAccruedEOY(...)` | Full-year accrual projection through Jan 1 |
| `calcTakenYTD(emp)` | Sum of `taken` entries and negative `adjustment` entries in the current PTO year |

### PTO Year Definition

The PTO year runs from **January 2 through January 1** of the following year. January 1 itself is treated as the last day of the prior PTO year (to handle the rollover transition cleanly). The function `getPTOYear(dateStr)` implements this logic.

### Rollover Cap

At year-end, any balance above 40 hours is forfeited. `calcNextRollover(emp)` returns `min(max(0, EOY balance), 40)`.

A rollover cap warning is shown to employees in the Balance view, but only between **October 1 and January 31**, to prevent alert fatigue during the rest of the year.

### PIN Security

PINs are hashed client-side using a simple 32-bit multiply-and-accumulate hash (`hashPin`). This is not cryptographically strong, but is sufficient for a low-stakes internal tool where the primary purpose is preventing casual access rather than protecting against targeted attacks. Hashed PINs are stored in Supabase — plaintext PINs are never persisted.

---

## Component Architecture

The app uses a flat, functional component structure. There is no component library or routing framework.

### View Layer (top-level screens)

```
App
├── HomeScreen          — Employee/admin login selector
├── LoginScreen         — PIN entry, PIN setup, PIN reset via hire date
├── EmployeeScreen      — Individual employee dashboard
│   ├── EmpTabBar
│   ├── Donut           — SVG donut chart (balance, EOY)
│   ├── Timeline        — Horizontal bar showing accrual/taken/rollover
│   ├── EntryForm       — Log taken, scheduled, or adjustment entries
│   ├── HistoryView     — Sortable entry list with optional accrual overlay
│   ├── ProjectedBalance — Future balance calculator
│   └── MiniCalendar    — Monthly calendar with PTO indicators
└── AdminScreen         — Admin dashboard
    ├── AdminTabBar
    ├── Overview tab    — Sortable staff table with key metrics
    ├── Staff tab       — Edit employees, manage entries, add new staff
    ├── Report tab      — ReportView — filterable report with CSV export
    ├── Calendar tab    — MiniCalendar (admin mode, shows all staff)
    └── Settings tab    — Admin PIN management, accrual policy display
```

### Shared/Primitive Components

| Component | Purpose |
|---|---|
| `Avatar` | Colored initials circle, hue derived deterministically from name |
| `Card` | Styled container div |
| `Field` | Label + input wrapper with optional hint text |
| `TopBar` | Page header with title, subtitle, and sign-out button |
| `SortTh` | Sortable table header cell |
| `useSort` | Custom hook for sortable table state |
| `Donut` | SVG donut chart |
| `Timeline` | Horizontal stacked bar chart with today marker |
| `MiniCalendar` | Monthly calendar; works in both employee and admin modes |
| `EntryForm` | PTO entry form (single day or date range) |
| `ChangePinModal` | Modal overlay for employee PIN changes |

---

## State Management

There is no external state management library. State lives in two places:

**App-level state** (in the `App` component):
- `staff` — the full array of employee objects, mirroring Supabase
- `adminPin` — the hashed admin PIN
- `view` — which top-level screen is active (`home | login | employee | admin`)
- `currentUser` — the name of the signed-in employee
- `ready` — whether the initial Supabase load has completed

**Local component state** — each screen manages its own UI state (active tab, form fields, sort column, etc.) using `useState`.

The `persist` function (defined in `App`, passed down as a prop) is the single write path to Supabase. Every component that needs to save data calls `persist(updatedStaffArray)`, which simultaneously updates Supabase and the local `staff` state.

---

## Data Flow

```
Browser loads index.html
  → React mounts App
  → useEffect fires → dbGet() → Supabase REST API
  → staff array and adminPin loaded into App state
  → HomeScreen renders with staff list

User selects name → LoginScreen
  → PIN verified client-side (hashPin comparison)
  → on success: currentUser set, view switches to EmployeeScreen

Employee logs PTO
  → EntryForm calls onSave()
  → EmployeeScreen calls persist(updatedStaff)
  → persist calls dbSet() → Supabase REST API (upsert)
  → App state updated → React re-renders

Admin edits employee
  → AdminScreen calls persist(updatedStaff)
  → same path as above
```

---

## Deployment

The app is hosted on GitHub Pages at:

```
https://cx-associates.github.io/pto-tracker
```

To update the app, edit `index.html` in the `cx-associates/pto-tracker` GitHub repository. GitHub Pages rebuilds automatically within ~60 seconds of a commit. No build step or CI pipeline is involved.

---

## Known Constraints

- **Single shared row**: All data for all employees is stored in one JSON blob. This is simple and sufficient for a small team, but would not scale to hundreds of employees or high write concurrency.
- **Client-side PIN hashing**: The hash function is not cryptographically strong. For a higher-security requirement, authentication should be moved to Supabase Auth or a backend service.
- **No offline support**: The app requires a network connection to load data. If Supabase is unreachable, the app loads with an empty staff list.
- **No audit trail**: Entry deletions and edits are not logged. The only historical record is the current state of the `entries` array.
