# Data Quality — Checks & Balances

Goal: every company that reaches the report is **real, current, correctly identified, and net-new**,
with sourced firmographics and honest confidence. Apply these on every run.

## 1. Website verification (Step 5b)
- ZoomInfo's domain can be a **stale legal-entity domain**. Confirm the brand's official site via
  WebSearch and correct the domain if it doesn't match the brand.
  - *Example:* ZoomInfo returned `lovemishas.com`; the live site is `mishaskindfoods.com`.
- Don't rely on a raw fetch to decide "broken" — this environment's egress frequently gets HTTP
  **403** even from healthy Shopify/e-comm sites. Use the search index as the liveness signal.
- Record `Website Verified` = "Yes — {date}" (or note the correction).

## 2. Company-is-alive check (automated via ZoomInfo)
- **Primary, automated gate:** drop any company with ZoomInfo `isDefunct: true` or a
  `companyStatus` in `settings.zoominfo.alive_check.drop_statuses` (e.g. `DEFUNCT_DOMAIN_DOWN`).
  Request `isDefunct` / `companyStatus` / `companyStatusDate` during enrichment and record the
  status in the report.
  - *Validated 2026-06-17:* Off The Cob → `isDefunct:true` / `DEFUNCT_DOMAIN_DOWN`; the 17 kept
    leads all returned `ALIVE`.
- **Backstop:** for young brands ZoomInfo may not have re-verified (stale `companyStatusDate`) or
  ambiguous status, search "<brand> out of business / closed / acquired" (caught Aurora Elixirs,
  winding down).

## 3. Identity / disambiguation
- Confirm the ZoomInfo record is **this** brand, not a same-named company (common for generic
  names like "Droplet", "Monday", "Drink"). Cross-check description + location + product.

## 4. Dedup integrity (see `dedup.md`)
- Match on **normalized root domain**, not raw `LIKE` (which over-matches). Verify the normalized
  roots are actually equal before excluding.
- Check **both** website and name, plus Leads, plus the **cross-run** master (`leads_master.csv`).
- Re-check after enrichment using the ZoomInfo id and any corrected domain.

## 5. Firmographic sanity checks
- **Revenue:** ZoomInfo understates DTC brands; treat figures as estimates and mark "unknown" when
  absent rather than guessing. Sanity-check revenue vs. employees vs. funding (e.g. a $2M brand
  with 500 employees is a red flag to re-check).
- **Geography:** confirm HQ is US/Canada (an ICP gate) — ZoomInfo occasionally lists a foreign
  parent (e.g. Real Coconut → SANA Foods resolved to Mexico).
- **Manufacturing model:** if inferred ("co-packer likely"), say so; don't assert it as confirmed.
- **Funding:** corroborate notable rounds with a second source (press / Crunchbase) before stating
  amounts and investors.

## 6. Confidence & flags
- Every row carries a **Tier** (Strong/Possible/Stretch) and a **Flags** column. Anything
  uncertain (publicly traded, possible retailer, possible in-house mfg, thin data) gets flagged
  rather than dropped or overstated.
- Prefer "unknown" over a guess. Quality over quantity.

## 7. Human-in-the-loop
- The summary is delivered as a **draft** (the connected Gmail MCP is draft-only), so a human
  reviews before anything is sent. The Google Sheet is the working artifact, not an auto-send.

## 8. Auditability
- Every run appends to `runs/log.md` (counts + the exact excluded/known + disqualified companies
  with reasons) and to `runs/leads_master.csv` (append-only). The trail makes errors traceable and
  prevents re-surfacing.

## 9. Salesforce safety
- **Read-only.** Dedup reads only; never write. ZoomInfo: enrich only what's needed (no mass export).

---
### Optional future hardening (not yet wired)
- Programmatic liveness check from an egress that isn't 403'd (e.g. a Salesforce/ZoomInfo-side
  enrichment status, or a allow-listed fetch) to auto-flag dead domains.
- Auto-write `ICP_Fit_Score__c` / notes back to Salesforce — **only** if/when write access is
  intentionally enabled (today SFDC is read-only by rule).
- Second-source corroboration of revenue via QuickBooks/Apollo/Crunchbase where available.
