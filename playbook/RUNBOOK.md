# DOSS Lead Search — Runbook

This is the procedure an agent (Claude) follows to produce one lead-search report. Follow the
steps in order. Read `config/icp.yaml`, `config/sources.yaml`, and `config/settings.yaml` first.

**Required integrations (MCP):** Salesforce (read-only), ZoomInfo, Web search/fetch, Gmail,
Google Drive.

**Goal:** ~`run.target_company_count` net-new, ICP-fit CPG companies that are **not** already in
Salesforce, each enriched with ZoomInfo firmographics, delivered as a Google Sheet + email.

---

## Step 0 — Load config & set the run date
- Parse the three config files. Note `recency_window_days`, `target_company_count`,
  `max_candidates_to_screen`, delivery targets, and dedup keys.
- Set `RUN_DATE` = today (America/Los_Angeles). Use it in the sheet title and run log.

## Step 1 — Confirm ICP gate
From `config/icp.yaml`, hold these in mind for screening:
- Revenue $10M–$200M core (engage up to $500M+); employees 10–500; HQ US/Canada.
- Physical-inventory consumer product (categories in `product_categories`).
- Outsources production (co-mans / 3PLs); on QuickBooks + spreadsheets or a broken stack.
- Disqualifiers: pure software/services, heavy in-house manufacturing, already on full ERP,
  complex WMS, and industries government / pharma-hospital / real estate / restaurants / VC.

## Step 1b — Apply current focus (if `config/focus.yaml: active`)
If a focus campaign is active, this run is biased toward a specific slice of the ICP (currently:
emerging, better-for-you / functional F&B brands that **look like the `anchors`** — Mezcla, De Soi).
- In `mode: boost`, surface lookalikes first but still allow other ICP-fit companies.
- In `mode: strict`, only report companies matching `focus.lookalike_profile`.
- Note the focus profile (categories, positioning keywords, stage, size) for use in Steps 2 & 6.
- The focus `size` band intentionally dips below the $10M ICP floor to catch brands early; ICP
  best-fit still scores highest (see scoring).

## Step 2 — Discover candidate companies
Build a raw candidate list (aim for up to `max_candidates_to_screen`).

**2a. Lookalike discovery via ZoomInfo (run FIRST when focus is active).** For each
`focus.anchors[].zoominfo_company_id`, call **ZoomInfo `find_similar_companies`** to get a ranked
list of companies that "look like" the anchor. Take the top matches, then proceed to dedup/enrich.
This is the most direct lookalike engine — start here when a focus is set.

**2b. Publications + web searches.** For each entry in `sources.yaml`:
- Use **WebSearch** for each `web_searches` query and **WebFetch** on `publications` URLs (and
  promising article links) to extract recently-mentioned companies within `recency_window_days`.
- For each candidate capture: **company name**, **website** (if shown), and a one-line
  **why-surfaced** note with the **source URL** (e.g. "Raised $12M Series A — Food Dive, 2026-05").
- Prefer brands tied to a *signal*: funding, product launch, retail expansion, new 3PL/warehouse.

**2c. X / Twitter (if `sources.yaml: x_discovery.enabled`).** No dedicated X MCP is connected, so
scan X through the web tools:
- Run each `x_discovery.searches` query with **WebSearch** (they're scoped with `site:x.com` /
  `site:twitter.com`), and **WebFetch** promising post URLs to extract the brand + claim.
- Also search the monitored `handles_to_monitor` and `hashtags` for recent funding/launch posts.
- Resolve each brand to a website before it enters the candidate list.

**2d. ZoomInfo discovery (if `sources.yaml: zoominfo_discovery.enabled`).** Complement the
publications by surfacing brands they miss:
- Use **ZoomInfo `search_scoops`** with `scoopTypes: [Funding]`, `publishedStartDate` =
  RUN_DATE − `scoop_window_days`, plus ICP company filters (industry, revenue, employees, US/CA).
  Use **`lookup`** to resolve valid industry/revenue/employee/state values first.
- Optionally **`search_companies`** within the ICP band for additional candidates.

**Normalize & de-duplicate the candidate list internally** by root domain
(see `playbook/dedup.md`). Drop anything in `exclude_domains` / `exclude_names`.

## Step 3 — Pre-screen against ICP (cheap, public info)
Before spending Salesforce/ZoomInfo calls, drop obvious non-fits using what you already know:
- Pure software/services, restaurant/food-service operator, pharma/hospital, government, real
  estate, VC firm → drop (note reason).
- Obviously global-only / non-US-CA, or clearly mega-cap (> $500M) → drop or flag.
Keep the rest as **screened candidates**.

## Step 4 — Salesforce dedup (exclude anything already known)
For each screened candidate, determine if it already exists in Salesforce. **Follow
`playbook/dedup.md` exactly.** In short:
- Normalize the candidate's root domain.
- Query Accounts by domain and by name; if `salesforce.check_leads`, also query Leads.
  Example SOQL (domain match):
  ```sql
  SELECT Id, Name, Website, Account_Status__c, OwnerId
  FROM Account
  WHERE Website LIKE '%acmebrand.com%'
  ```
  And a name/SOSL fallback (see dedup.md).
- If **any** match is found (any status, incl. churned/closed-lost/disqualified), mark the
  candidate **KNOWN** and exclude it. Record the matched Account Id + status in the run log.
- **Cross-run dedup (`settings.dedup_across_runs`):** also drop any candidate already present in
  `runs/leads_master.csv` (reported in a prior run). The master CSV + `runs/log.md` are the
  pipeline's memory so 3-hourly runs don't re-surface the same companies.
- Keep only **net-new** candidates. Continue until you have enough to reach the target after
  enrichment; loop back to Step 2 if the net-new pool is too small.

## Step 5 — Enrich net-new candidates with ZoomInfo
- Call **ZoomInfo `enrich_companies`** in batches of up to 10, identified by `domain` (preferred)
  or `companyName`. Request `zoominfo.enrich_fields` from settings.
- If `zoominfo.pull_funding_scoops`, also call **`enrich_scoops`** with `scoopTypes: [Funding]`
  to capture round type, amount, date, and investors.
- Request the `zoominfo.enrich_fields` including **`isDefunct`, `companyStatus`,
  `companyStatusDate`** — these drive the automated alive-check in Step 5b.
- Capture the **ZoomInfo company id** for each (used for the sheet + future dedup).
- If ZoomInfo returns no match, keep the candidate but mark firmographics "Not in ZoomInfo" and
  fill what you can from the publication/web source.

## Step 5b — Verify website + that the company is a live, matching business
Data-accuracy gate before anything goes in the report (see `playbook/data_quality.md`). For each
enriched candidate:
- **Confirm the working website.** ZoomInfo's domain is sometimes a stale legal-entity domain
  (e.g. it returned `lovemishas.com` for Misha's Kind Foods, whose live site is
  `mishaskindfoods.com`). Verify with **WebSearch** that the brand's official site resolves and the
  domain matches the brand; correct the domain if it doesn't. (Note: WebFetch/curl often get HTTP
  403 from this environment's egress even for healthy sites, so use the search index as the signal,
  not a raw fetch.)
- **Automated alive-check (`zoominfo.alive_check`).** Drop any company with ZoomInfo
  `isDefunct: true` or a `companyStatus` in `drop_statuses` (e.g. `DEFUNCT_DOMAIN_DOWN`). Validated
  2026-06-17: Off The Cob → `DEFUNCT_DOMAIN_DOWN`/`isDefunct:true`; the 17 kept leads → `ALIVE`.
  Record `companyStatus` in the report. This is the first, cheap gate.
- **Backstop the status with a quick check** for borderline/younger brands ZoomInfo may not have
  re-verified — search "<brand> out of business / closed" if `companyStatusDate` is stale or status
  is ambiguous (e.g. Aurora Elixirs was winding down).
- **Confirm brand ↔ firmographics match** (the ZoomInfo record is actually this brand, not a
  same-named company). 
- Record the outcome in the **Website Verified** column ("Yes — {date}", or note a correction).
  Anything you can't verify → flag it and lower confidence rather than shipping it silently.

## Step 6 — Score & finalize
- Apply the rubric in `playbook/scoring.md` to compute an **ICP fit score (0–100)**, a **tier**
  (Strong / Possible / Stretch), a short **rationale**, and any **disqualifier flags**.
- Re-confirm the dedup result post-enrichment using the ZoomInfo id and any corrected domain
  (a company can resolve to a different primary domain after enrichment).
- Sort by `run.sort_by` and take the top `target_company_count`. Keep a few flagged "Stretch"
  entries only if the strong pool is short.

### Step 6b — No-op when nothing is new (`settings.schedule.skip_report_when_no_new`)
If after dedup there are **zero net-new** companies (common on frequent 3-hourly runs), **stop
here**: do not rebuild the sheet, do not email. Just append a one-line "no net-new" entry to
`runs/log.md`. This keeps frequent runs cheap and quiet.

## Step 7 — Update the master tracker (single sheet, sectioned by run)
The report is **one master sheet** (`delivery.google_drive.master_sheet_title`), with each run as
its own dated section, newest on top. The Google Drive MCP **cannot edit a Sheet in place**, so
rebuild it from the cumulative dataset:
1. **Append** this run's rows to `runs/leads_master.csv` (the append-only source of truth in git),
   tagging each row with its `Run Date` and a section label.
2. In the Drive folder (`delivery.google_drive.folder_id`), **find the existing master sheet**
   (search by title). 
3. **Recreate** the master sheet from the full `leads_master.csv`, laid out as dated sections
   (see `templates/report_sheet_schema.md` → "Master tracker layout"): a section banner row
   (`=== Run: {RUN_DATE} — {N} net-new ({focus}) ===`), then the column header row, then the run's
   rows; repeat for each prior run below, newest first.
4. **Remove the prior master sheet** so only one remains. NOTE: the current Drive MCP exposes no
   delete/trash tool, so if you can't delete programmatically, leave the prior copy and tell Max to
   delete the older "Master Tracker" copy (sort the folder by modified date — keep the newest).
5. Capture the new file URL, but tell Max to **bookmark the folder** (stable) rather than the file
   (its URL changes each rebuild). 

## Step 8 — Email the summary
- Only if there were net-new companies this run. Using the Gmail MCP, send (or draft — the
  connected Gmail MCP currently supports **drafts only**, so create a draft and tell Max) to
  `delivery.email.to` an email from `templates/email_summary_template.md`: counts, the **top 5**
  new companies with one-line rationales, the **master sheet link**, and the **folder link**.
- Subject: `{subject_prefix} {N} net-new ICP companies — {RUN_DATE}`.

## Step 9 — Log the run
- Append a line to `runs/log.md`: date, # scanned, # net-new, # reported, sheet URL, and the list
  of KNOWN exclusions (name + matched Account Id) so we have an audit trail and avoid re-work.

---

## Guardrails
- **Never write to Salesforce** — it is read-only and this pipeline only *reads* to dedup.
- **ZoomInfo compliance:** individual B2B research only; do not mass-export. Enrich only the
  net-new candidates needed for the run.
- **Be honest about gaps:** if revenue/funding isn't available, say "unknown" rather than guess.
- **Respect the target size:** quality over quantity — better 15 strong fits than 25 noisy ones.
