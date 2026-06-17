# ICP Fit Scoring Rubric

Produce an **ICP fit score (0–100)**, a **tier**, a short **rationale**, and **disqualifier
flags** for each net-new, enriched company. The score guides ordering; the rationale is what makes
the report useful to an AE.

## Scoring (additive, capped at 100)

| Dimension | Points | How to award |
|-----------|-------:|--------------|
| **Product category** | 25 | Full 25 if a physical-inventory consumer category in `icp.yaml` (food & bev, beauty, supplements, home goods, pet, apparel, consumer electronics, personal care, health & wellness). 0 if not a physical product. |
| **Revenue fit** | 20 | 20 if $10M–$200M. 12 if $200M–$500M (stretch). 8 if just under $10M but fast-growing. 0 if >$500M or clearly tiny/pre-revenue. |
| **Employee fit** | 10 | 10 if 10–500. 5 if just outside. 0 if far outside. |
| **Geography** | 10 | 10 if HQ US/Canada. 0 otherwise. |
| **Outsourced manufacturing** | 15 | 15 if evidence of co-man / 3PL use. 7 if unknown but plausible (small/mid brand). 0 if heavy in-house manufacturing/warehousing. |
| **System / pain signal** | 10 | 10 if on QuickBooks + spreadsheets / broken stack / no ERP. 5 if unknown. 0 if already on a full ERP (NetSuite, Intacct, Dynamics, Epicor, Acumatica, SAP). |
| **Momentum signal** | 10 | 10 if recent funding / launch / retail or 3PL expansion (the reason it surfaced). 5 if older signal. 0 if none. |

## Current-focus boost (if `config/focus.yaml: active`)
When a focus campaign is active, bias ordering toward the lookalike profile (currently: emerging
better-for-you / functional F&B brands like Mezcla & De Soi):
- **+15 lookalike bonus** if the company matches `focus.lookalike_profile` (category + positioning
  keywords + recently-funded stage). This can push a promising sub-$10M emerging brand above a
  larger but less-relevant ICP company.
- In `mode: strict`, **exclude** anything that doesn't match the lookalike profile.
- A sub-$10M anchor-style brand that is fast-growing and recently funded should land **Possible**
  or better even though it's under the core revenue band — note "below ICP floor, early-stage" in
  the rationale.

## Tiers
- **Strong fit:** score ≥ 70 **and** no hard disqualifier.
- **Possible fit:** 50–69, or strong but with one unknown dimension.
- **Stretch:** 30–49, or outside core band but worth a look (e.g. $200M–$500M, or category-adjacent).
- **Disqualified:** any hard disqualifier present → exclude from the report (log the reason).

## Hard disqualifiers (override score → exclude)
From `icp.yaml`:
- No physical inventory (pure software/services).
- Heavy in-house manufacturing & warehousing, minimal outsourcing.
- Already on / actively requires full ERP, or needs complex WMS build-out.
- Industry: government, pharma/hospitals, real estate, restaurants, VC firm.
- HQ outside US/Canada.

## Rationale (1–2 sentences per company)
State *why it fits and why now*. Good example:
> "$30M better-for-you snack brand, ~80 employees, uses co-packers, just raised an $8M Series A to
> expand into 2,000 Target doors — classic QuickBooks-and-spreadsheets scaling pain. Strong fit."

If a dimension is unknown, say so ("manufacturing model unconfirmed") rather than inventing it.
