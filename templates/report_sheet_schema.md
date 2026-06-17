# Report — Google Sheet Schema

One sheet per run, titled `DOSS Lead Search — {RUN_DATE}`, saved in the Drive folder from
`settings.yaml`. Header row first, then one row per company, sorted by ICP fit score (desc).

## Columns (in order)

| # | Column header | Contents |
|---|---------------|----------|
| 1 | `Company` | Company / brand name |
| 2 | `Website` | Root domain or full URL |
| 3 | `ICP Fit Score` | 0–100 (see scoring.md) |
| 4 | `Tier` | Strong / Possible / Stretch |
| 5 | `Why Surfaced` | Signal + source, e.g. "Series A $12M — Food Dive 2026-05" |
| 6 | `Source URL` | Link to the article / search result |
| 7 | `Product Category` | e.g. Food & Beverage, Beauty, Supplements |
| 8 | `Product Type / Description` | Short description of what they make |
| 9 | `Est. Revenue` | From ZoomInfo (range or figure); "unknown" if absent |
| 10 | `Employees` | From ZoomInfo; "unknown" if absent |
| 11 | `HQ City` | ZoomInfo |
| 12 | `HQ State` | ZoomInfo |
| 13 | `HQ Country` | ZoomInfo (expect US/Canada) |
| 14 | `Founded` | Year founded (ZoomInfo) |
| 15 | `Total Funding` | ZoomInfo total funding amount |
| 16 | `Last Round` | Round type + amount + date (ZoomInfo scoops) |
| 17 | `Investors` | Named investors if available |
| 18 | `Manufacturing Model` | Co-man/3PL/in-house/unknown |
| 19 | `Current Systems` | QuickBooks/spreadsheets/ERP/unknown (if known) |
| 20 | `Rationale` | 1–2 sentence "why fit + why now" |
| 21 | `Disqualifier Flags` | Any soft concerns (blank if none) |
| 22 | `ZoomInfo Company ID` | For future dedup / SFDC import |
| 23 | `LinkedIn URL` | If available |
| 24 | `Date Added` | RUN_DATE |
| 25 | `Website Verified` | "Yes — {date}" once the site is confirmed live + brand-matched (Step 5b); note any correction |
| 26 | `Company Status` | ZoomInfo `companyStatus` + date, e.g. "ALIVE (2026-06-02)" — from the automated alive-check (Step 5b). Defunct companies are dropped, not listed. |

## Master tracker layout (sectioned by run)
The deliverable is a **single** sheet titled `DOSS Lead Search — Master Tracker`, accumulating
every run as a dated section, newest on top:

```
=== Run: 2026-06-20 — 4 net-new (Mezcla/De Soi lookalikes) ===
Company, Website, ICP Fit Score, ... (full header row)
<this run's rows>

=== Run: 2026-06-17 — 18 net-new (Mezcla/De Soi lookalikes) ===
Company, Website, ICP Fit Score, ... (full header row)
<that run's rows>
```

- The append-only source of truth is `runs/leads_master.csv` (in git). Each run appends its rows
  there first, then the Drive master sheet is rebuilt from it (the Drive MCP can't append in place).
- A section banner is a single row whose first cell is `=== Run: {DATE} — {N} net-new ({focus}) ===`.
- Repeat the column header row inside each section so each section is independently sortable.

## Build notes
- Easiest path with the Drive MCP: assemble the rows as CSV and create the file with
  `content_mime_type: text/csv` (it auto-converts to a Google Sheet), or create a
  `application/vnd.google-apps.spreadsheet` and populate. Verify the file opens as a Sheet.
- Keep values clean (no embedded newlines in cells; use "; " to separate investors).
- Always include the header row exactly as above so the Sheet is filter/sort-ready.
