# Housing.com — per-site comparison (run `2026-06-18-housing-blr-1bhk-furnished-bachelors`)

**Filters:** Bengaluru · Rent · 1 RK + 1 BHK · ₹12,000–22,000 · Fully Furnished · Lease type = Bachelor.
**Captured:** 2026-06-18 by Cowork (Claude in Chrome) from the deep-linked filtered search
`https://housing.com/rent/search-C3G1L4P38f9yfbk7p3m2h1fT99cUgz4`.
**Set size:** **328 unique listings** captured and analysed. (Housing's header claims "392 properties", but the last ~3 page-advances returned only repeats/promoted cards — the deduplicated in-budget set is 328. See `findings.md`.)

> **Gender requirement.** The user asked for bachelor listings open to **both men and women** (not single-gender; "all" acceptable). Housing.com's lease-type filter has a **single "Bachelor" option with no male/female split**, and the per-listing lease set (Family/Company/Bachelor) carries no gender flag. So there is no single-gender restriction to exclude — **all 328 bachelor-eligible listings satisfy the requirement.** [Inference — based on the filter UI and recipe; Housing exposes no gender field. Expected, not a guarantee about every listing's off-platform preference.]

## How the numbers were produced
- `calc_price_per_sqft` = `rent ÷ area` (built-up area as captured), ₹/sq.ft/month.
- `calc_true_monthly_cost` = `rent + maintenance(if extra) + (brokerage + move_in)/12 + deposit×0.005` (STAY_MONTHS=12; deposit charged at 0.5%/mo opportunity cost). Per `docs/calculations.md`.
- `calc_cost_basis` = **full** when maintenance, brokerage and move-in are all known (146 of 328); **lower-bound** when ≥1 is unknown (182 of 328). Compare `full` rows to `full` rows.
- All ₹/sq.ft figures are on **built-up area** (Housing's card basis) — not directly comparable to carpet-area ₹/sq.ft.

## Summary (all 328)

| Metric | Value |
|---|---|
| Listings | 328 (1 BHK: 214 · 1 RK: 114) |
| Listed by | Owner: 96 · Broker/agent: 232 |
| Rent ₹/month | min 12,000 · median 19,000 · mean 18,442 · max 22,000 |
| ₹/sq.ft/month (built-up) | min 9 · median 34.1 · mean 37.3 · max 112.5 |
| Complete-cost (`full` basis) | 146 of 328 |

### By unit type
| Unit | Count | Median rent | Median ₹/sq.ft |
|---|---|---|---|
| 1 BHK | 214 | 20,000 | 30.0 |
| 1 RK | 114 | 16,000 | 46.1 |

### By property type
| Type | Count | Median rent | Median ₹/sq.ft |
|---|---|---|---|
| Apartment | 223 | 19,000 | 33.3 |
| Independent floor | 58 | 19,000 | 37.2 |
| Independent house | 30 | 16,250 | 29.5 |
| Studio | 16 | 16,000 | 47.6 |
| Penthouse | 1 | 20,000 | 66.7 |

### By lister
| Lister | Count | Median rent | Median ₹/sq.ft |
|---|---|---|---|
| Owner | 96 | 18,000 | 33.0 |
| Broker/agent | 232 | 19,150 | 34.7 |

### Top localities by listing count
| Area | Count | Median rent |
|---|---|---|
| BTM Layout | 75 | 19,500 |
| (Bengaluru — area not specified) | 40 | 19,000 |
| S.G. Palya | 27 | 20,500 |
| Whitefield | 14 | 21,000 |
| Electronic City | 13 | 18,000 |
| Koramangala | 10 | 17,300 |
| Doddanekundi | 9 | 15,499 |
| Marathahalli | 9 | 20,000 |
| Kadubeesanahalli | 9 | 19,000 |
| BTM Layout 2nd Stage | 8 | 20,000 |
| Indira Nagar | 8 | 17,750 |
| Brookefield | 6 | 15,500 |

## Best value — lowest true monthly cost (complete-cost `full` rows only)

`tmc` = effective ₹/month incl. amortised one-time costs + deposit opportunity cost.

| # | Locality | Unit | Area (sq.ft) | Rent | Deposit | ₹/sq.ft | True ₹/mo | By | Link |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Stage 2, ISRO Layout | 1 RK | 300 | 12,000 | 50,000 | 40.0 | 13,250 | Owner | [link](https://housing.com/rent/20486367-300-sqft-1-rk-studio-on-rent-in-isro-layout-bengaluru) |
| 2 | Rachenahalli, Jakkur | 1 BHK | 450 | 12,500 | 100,000 | 27.8 | 14,042 | Owner | [link](https://housing.com/rent/20313180-450-sqft-1-bhk-independent-house-on-rent-in-jakkur-bengaluru) |
| 3 | Seegehalli, K R Puram | 1 BHK | 600 | 13,500 | 50,000 | 22.5 | 14,875 | Owner | [link](https://housing.com/rent/20496157-600-sqft-1-bhk-independent-house-on-rent-in-k-r-puram-bengaluru) |
| 4 | Stage 2, Nagarbhavi | 1 BHK | 750 | 13,200 | 100,000 | 17.6 | 14,950 | Owner | [link](https://housing.com/rent/20449323-750-sqft-1-bhk-independent-house-on-rent-in-nagarbhavi-bengaluru) |
| 5 | Someshwara Layout, Bhoganhalli | 1 RK | 450 | 13,000 | 30,000 | 28.9 | 15,317 | Broker | [link](https://housing.com/rent/16816213-450-sqft-1-rk-studio-on-rent-in-bhoganhalli-bengaluru-v3) |
| 6 | Stage 1, BTM Layout | 1 RK | 400 | 13,500 | 30,000 | 33.8 | 15,900 | Broker | [link](https://housing.com/rent/20418367-400-sqft-1-rk-apartment-on-rent-in-btm-layout-bengaluru) |
| 7 | Stage 1, BTM Layout | 1 RK | 500 | 13,500 | 40,000 | 27.0 | 15,950 | Broker | [link](https://housing.com/rent/20321640-500-sqft-1-rk-apartment-on-rent-in-btm-layout-bengaluru) |
| 8 | Stage 1, BTM Layout | 1 RK | 400 | 13,600 | 30,000 | 34.0 | 16,017 | Broker | [link](https://housing.com/rent/20454818-400-sqft-1-rk-apartment-on-rent-in-btm-layout-bengaluru) |
| 9 | Stage 1, BTM Layout | 1 RK | 500 | 13,600 | 35,000 | 27.2 | 16,042 | Broker | [link](https://housing.com/rent/20402514-500-sqft-1-rk-apartment-on-rent-in-btm-layout-bengaluru) |
| 10 | Gandhi Nagar, Munnekollal | 1 RK | 350 | 12,999 | 30,000 | 37.1 | 16,316 | Broker | [link](https://housing.com/rent/20469461-350-sqft-1-rk-apartment-on-rent-in-munnekollal-bengaluru) |
| 11 | Amrutahalli | 1 RK | 550 | 15,000 | 50,000 | 27.3 | 16,500 | Broker | [link](https://housing.com/rent/20409456-550-sqft-1-rk-apartment-on-rent-in-amrutahalli-bengaluru) |
| 12 | Doddanekundi | 1 RK | 250 | 12,999 | 25,998 | 52.0 | 16,546 | Broker | [link](https://housing.com/rent/20300409-250-sqft-1-rk-apartment-on-rent-in-dodda-nekkundi-bengaluru) |

## Best value — lowest ₹/sq.ft (built-up; complete-cost `full` rows)

Favours larger units; treat the very low figures as the site's stated built-up area (captured as shown, not independently verified).

| # | Locality | Unit | Type | Area | Rent | ₹/sq.ft | By |
|---|---|---|---|---|---|---|---|
| 1 | Kaverappa Layout, Kadubeesanahalli | 1 RK | apartment | 1,500 | 15,000 | 10.0 | Owner |
| 2 | Sai Colony, K R Puram | 1 BHK | independent house | 1,300 | 18,000 | 13.9 | Owner |
| 3 | Stage 2, BTM Layout 2nd Stage | 1 BHK | independent house | 1,000 | 15,500 | 15.5 | Broker |
| 4 | Carmelaram | 1 RK | independent floor | 1,200 | 19,000 | 15.8 | Broker |
| 5 | Stage 2, Nagarbhavi | 1 BHK | independent house | 750 | 13,200 | 17.6 | Owner |
| 6 | Stage 2, BTM Layout 2nd Stage | 1 BHK | apartment | 960 | 18,000 | 18.8 | Broker |
| 7 | Stage 1, BTM Layout | 1 BHK | independent floor | 900 | 19,000 | 21.1 | Broker |

## Caveats (per the schema/rules — captured as shown, not "corrected")
- **`contact_phone`** is login/lead-gated on every listing → left blank (a gap, not worked around).
- **`floor`** and the **exact per-listing lease slash-set** are detail-page-only and not in the list data; left blank. The Bachelor filter guarantees bachelor-eligibility for all rows (recorded as `tenant_preference = Bachelor`).
- **`building_name`** left blank: the address sometimes carries a society name and sometimes the broker's firm name in the same position, so it wasn't reliably separable. Full `address` is preserved.
- **Area basis** is built-up; some listings show very large built-up areas (e.g. a "1 RK" at 1,500 sq.ft) — captured verbatim, flagged here, not adjusted.
- **Cost completeness:** 182 of 328 are `lower-bound` (a cost field — usually move-in/painting — wasn't exposed), so their true monthly cost understates reality; the shortlists above use only `full`-basis rows.
