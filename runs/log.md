# Run Log

| Run Date | Scanned | Net-new | Reported | Sheet | Notes |
|----------|--------:|--------:|---------:|-------|-------|
| 2026-06-17 (run 2) | ~45 | 9 | 9 | [Master Tracker](https://docs.google.com/spreadsheets/d/1TPOSYzahiZ9sqsclm__viQRupK8KnpSzmfuCisW2Cjo/edit) | Mezcla-profile, FUNDED, 20+ employees. find_similar pool exhausted → pivoted to ZoomInfo search_companies (US, F&B, 21-200 emp, fundingAmountMin, ≤$200M). All in-band ICP. ~36 candidates excluded as already-in-SFDC (heavy saturation). All 9 ALIVE. Email draft created. |
| 2026-06-17 | 32 | 17 | 17 | [Master Tracker](https://docs.google.com/spreadsheets/d/1vzlc3MMp3gJA2mvOrvdPhxnNIUjCEUg4t0SIXQ_8ASg/edit) | Lookalike-led run (Mezcla + De Soi). Email draft created for max@doss.com. Website verification: removed Off The Cob (defunct); corrected Misha's domain → mishaskindfoods.com (18→17). ZoomInfo alive-check validated: 17 kept = ALIVE; Off The Cob = isDefunct/DEFUNCT_DOMAIN_DOWN. Added Company Status column. |

### 2026-06-17 (run 2) — details
Target: Mezcla-profile CPG, **funded + 20+ employees**, net-new.
Method: ZoomInfo `find_similar_companies` (Mezcla) pool was exhausted for these filters (unfunded
family Latin-food makers / co-packers / already-in-SFDC; only Motif had real funding + >20 emp and
it **wound down in 2024** — caught by the web backstop). Pivoted to ZoomInfo `search_companies`:
country US, industry Food & Beverage, employeeRangeMin 21 / max 200, fundingAmountMin 5000,
revenue ≤ $200M. ~45 curated F&B brands screened; the vast majority were already in SFDC (heavy
saturation on known funded better-for-you brands). All 9 net-new passed the ALIVE check.

REPORTED (9, all in-band $11M–$38M): Starday, Cheribundi, Wicked Kitchen, Tomorrow Farms,
Black Sheep Foods, Rebellyous Foods, Tender Food, True Essence Foods, Nature's Fynd.

### 2026-06-17 — details
Focus: `config/focus.yaml` lookalikes of Mezcla (`481310726`) and De Soi (`566126302`).
Discovery: ZoomInfo `find_similar_companies` on both anchors → curated 32 candidates → enriched → deduped.

EXCLUDED (already in SFDC, 11):
- Ghia → Account 001WR000011Hm5aYAC (Closed Lost)
- Bonbuz → 001WR000017LR2JYAW (Closed Lost)
- Hippeas → 001WR000017M0BLYA0 (Early Funnel)
- HOP WTR → 001WR000015lwunYAA (No Funnel Activity) [+ dup 001WR000016E66TYAS "HOPWTR"]
- Spiritless → 001WR000017KeMeYAK (No Funnel Activity)
- PeaTos → 001WR000017LfwbYAC (No Funnel Activity)
- gimme Seaweed → 001WR000014V5K1YAK (No Funnel Activity)
- Mocktail Club → 001WR00001KbBDfYAN (No Funnel Activity)
- Rind Snacks → 001WR00001CfCllYAF (No Funnel Activity — owned by Max)
- Mingle Mocktails → 001WR000012WrkIYAS (Closed Lost)
- Heywell → 001WR00001NsziIYAR (No Funnel Activity)

DISQUALIFIED (3): Aurora Elixirs (winding down), Real Coconut/SANA Foods (HQ Mexico), For All Drinks (media/events, not a product brand).

REPORTED (18): Droplet, New Brew, Figlia, Casamara Club, Optimist Drinks, Rock Grace, MASA Chips, Ohza, Sarilla, Misha's Kind Foods, TÖST, Starla, Yerbae, Off The Cob, Tasty Brand, Vegan Dude Foods, Sèchey, Blue Moose.

<!--
Append one row per run. Below each row, optionally list the KNOWN exclusions:
  EXCLUDED (already in SFDC): <Company> -> Account <Id> (<Status>); ...
-->
