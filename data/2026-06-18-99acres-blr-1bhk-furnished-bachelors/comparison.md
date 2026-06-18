# 99acres — Bengaluru 1 BHK, ₹12k–22k, fully furnished, bachelors (men & women / all)

**Run:** `2026-06-18-99acres-blr-1bhk-furnished-bachelors` · captured 2026-06-18 · source: 99acres.com

## Coverage & method (read first)

- **350 listings captured** out of **493** that matched the filters on the live site (1 RK/1 BHK · ₹12,000–22,000 · Furnished · Available-for Single Women + Single Men). That is **pages 1–15 of 20**; pages 16–20 (~143 listings) were **not captured** — the results list started returning empty skeletons (rate-limit) after rapid pagination, and a later detail-page sweep tripped a **CAPTCHA** anti-bot check. Neither was bypassed (per capture rules).
- **Tenant rule applied via two signals, not full per-listing confirmation.** 99acres only shows a listing's tenant preference on its *detail* page. The search filter already restricts results to bachelor-eligible (Single Women ∪ Single Men). Of a 3-listing detail sample, all read **"Available For: All"**. Every listing's description was scanned for single-gender wording; **4** were flagged (below). Detail-page confirmation of those 4 was blocked by the CAPTCHA, so their gender flags are **[Inference], not verified**.
- **1 RK studios are included and flagged** (your choice). 99acres bundles "1 RK / 1 BHK" into one filter, so **110** of the 350 rows are 1 RK studios, not true 1 BHK.

## What the set looks like

- **True 1 BHK:** 240  ·  **1 RK studios (flagged):** 110
- **Matches your 'both men & women / all' rule:** 347  ·  **Single-gender (flagged, excluded from that set):** 3
- **Area basis:** carpet 144, built-up 69, super built-up 83, plot 54 (plot = land area, not living area — ₹/sqft on those is understated)
- **Posted by:** owner 150, dealer 58, builder 37, unlabelled 105
- **Cost coverage:** 99acres exposes only **rent** and (sometimes) **deposit**. Maintenance, brokerage and move-in charges are not shown, so `calc_true_monthly_cost` is a **lower bound** for every row.

## Single-gender listings flagged (excluded from the both/all set) — [Inference], unverified

| listing_id | building / locality | rent | flag (from description) |
|---|---|---|---|
| A91757048 | CV Raman Nagar, Old Madras Road, Bangalore East — CV Raman Nagar | ₹21,000 | men-only |
| K90111822 | SL Ladies PG — Bellandur | ₹22,000 | women-only |
| W51505944 | Jayanagar, Bangalore, Bangalore South — Jayanagar | ₹20,000 | women-only |
| H89355255 | Neeladri Nagar, Bangalore, Bangalore South — Neeladri Nagar | ₹21,000 | both phrasings — review |

## Lowest ₹/sqft — true 1 BHK, living-area basis (carpet/built-up/super), rule-matching

Sorted by ₹/sqft/month ascending. Excludes 1 RK studios, plot-area listings, single-gender-flagged, and listings with implausible areas (kept band 250–900 sqft to filter poster-error giant areas). Top 20 shown; full set is in the CSV.

| listing_id | locality | type | area (basis) | rent | deposit | ₹/sqft | posted |
|---|---|---|---|---|---|---|---|
| O91446354 | Ananth Nagar Phase 2 | apartment | 800 (super) | ₹13,500 | ₹85,000 | 16.88 | owner |
| K91920752 | A.S. Raju Nagar | apartment | 650 (carpet) | ₹12,000 | — | 18.46 | owner |
| Q91026462 | ST Bed | apartment | 700 (carpet) | ₹13,000 | ₹2,600 | 18.57 | owner |
| J82162400 | BTM 2nd Stage | apartment | 900 (carpet) | ₹17,000 | ₹60,000 | 18.89 | dealer |
| A91544750 | Kadubeesanahalli | apartment | 840 (carpet) | ₹16,000 | — | 19.05 | owner |
| H91715128 | Thanisandra | apartment | 680 (carpet) | ₹14,000 | ₹28,000 | 20.59 | owner |
| S91714088 | Whitefield | apartment | 720 (super) | ₹15,000 | — | 20.83 | owner |
| H39856551 | Sachidananda Nagar | apartment | 800 (super) | ₹17,000 | ₹100,000 | 21.25 | owner |
| S90813534 | Nimbekaipura | apartment | 700 (carpet) | ₹15,000 | ₹100,000 | 21.43 | owner |
| O79648501 | Electronics City Phase 1 | builder floor | 600 (carpet) | ₹13,000 | ₹26,000 | 21.67 | builder |
| T91963948 | Whitefield | serviced apartment | 690 (super) | ₹15,000 | — | 21.74 | owner |
| B91818210 | ST Bed | apartment | 660 (super) | ₹15,000 | — | 22.73 | owner |
| M88325576 | RK Hegde Nagar | independent house | 850 (built-up) | ₹20,000 | — | 23.53 | owner |
| Q58444016 | Electronics City Phase 1 | apartment | 800 (carpet) | ₹19,000 | ₹26,000 | 23.75 | owner |
| K91751238 | Srirampura Jakkur | apartment | 900 (carpet) | ₹21,500 | ₹107,500 | 23.89 | owner |
| V90393404 | Shikaripalya | apartment | 500 (super) | ₹12,000 | ₹40,000 | 24.00 | owner |
| V41201513 | BTM Layout | apartment | 900 (built-up) | ₹21,900 | — | 24.33 | — |
| X91955282 | Brigade Road | apartment | 610 (super) | ₹15,000 | — | 24.59 | owner |
| V41470355 | BTM 2nd Stage | apartment | 850 (built-up) | ₹20,900 | ₹40,900 | 24.59 | — |
| E46491785 | Koramangala | apartment | 879 (super) | ₹22,000 | ₹44,000 | 25.03 | — |

## Caveats

- ₹/sqft is computed on each listing's **own** area basis (carpet vs built-up vs super) — not strictly comparable across rows. Plot-area listings are excluded from the ₹/sqft table for that reason.
- `tenant_preference` is blank for all but 3 detail-confirmed rows; treat the column as confirmed-only.
- Deposits given as "N months rent" were computed (N × rent) and noted; a few cards show implausible deposits (₹1, ₹40) — kept as captured and flagged, not corrected.
- Some poster-entered values are noisy (e.g. plot 13,500 sqft on a 1 BHK builder floor); captured as shown.