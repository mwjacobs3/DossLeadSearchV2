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
- Keep only **net-new** candidates. Continue until you have enough to reach the target after
  enrichment; loop back to Step 2 if the net-new pool is too small.

## Step 5 — Enrich net-new candidates with ZoomInfo
- Call **ZoomInfo `enrich_companies`** in batches of up to 10, identified by `domain` (preferred)
  or `companyName`. Request `zoominfo.enrich_fields` from settings.
- If `zoominfo.pull_funding_scoops`, also call **`enrich_scoops`** with `scoopTypes: [Funding]`
  to capture round type, amount, date, and investors.
- Capture the **ZoomInfo company id** for each (used for the sheet + future dedup).
- If ZoomInfo returns no match, keep the candidate but mark firmographics "Not in ZoomInfo" and
  fill what you can from the publication/web source.

## Step 6 — Score & finalize
- Apply the rubric in `playbook/scoring.md` to compute an **ICP fit score (0–100)**, a **tier**
  (Strong / Possible / Stretch), a short **rationale**, and any **disqualifier flags**.
- Re-confirm the dedup result post-enrichment using the ZoomInfo id and any corrected domain
  (a company can resolve to a different primary domain after enrichment).
- Sort by `run.sort_by` and take the top `target_company_count`. Keep a few flagged "Stretch"
  entries only if the strong pool is short.

## Step 7 — Build the Google Sheet
- Find or create the Drive folder `delivery.google_drive.folder_name` (search via Drive MCP;
  create with mimeType `application/vnd.google-apps.folder` if missing).
- Create a Google Sheet titled `DOSS Lead Search — {RUN_DATE}` in that folder. Use the exact
  columns from `templates/report_sheet_schema.md`, header row first, one row per company.
  (Create as `text/csv` content converting to a Google Sheet, or a Google Sheet file — see
  template notes.)
- Capture the resulting **file URL**.

## Step 8 — Email the summary
- Using the Gmail MCP, send to `delivery.email.to` an email built from
  `templates/email_summary_template.md`: counts (scanned / net-new / reported), the **top 5**
  companies with one-line rationales, and the **link to the Sheet**.
- Subject: `{subject_prefix} {N} net-new ICP companies — {RUN_DATE}`.
- Default to **sending**; if the session is configured for review, create a **draft** instead and
  tell Max.

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
