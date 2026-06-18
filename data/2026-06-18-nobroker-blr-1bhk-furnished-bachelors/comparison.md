# NoBroker — per-site comparison (run `2026-06-18-nobroker-blr-1bhk-furnished-bachelors`)

**Filters:** Bengaluru · Rent · 1 BHK (`type=BHK1`, 1 RK excluded) · ₹12,000–22,000 · Fully Furnished · tenant = bachelor-eligible (`leaseType=BACHELOR_MALE,BACHELOR_FEMALE`), kept only listings open to **both** men and women (or any).
**Captured:** 2026-06-18 by Cowork (Claude in Chrome) via NoBroker's filtered results API (`/api/v3/multi/property/RENT/filter`), per-locality (NoBroker has no city-wide search — see `docs/sites/nobroker.md`).
**Localities searched (8):** BTM Layout, Koramangala, HSR Layout, Marathahalli, Whitefield, Bellandur, Electronic City, Indiranagar — each searched separately (`radius=2.0 km`) and combined.
**Set size:** **50 unique listings** kept. 56 listings matched the budget/BHK/furnishing filter across the 8 searches; **5 were excluded as single-gender** (see below) and **1 was an internal duplicate**; leaving 50.

> **Gender requirement.** The user asked for listings open to **both men and women** bachelors (single-gender not acceptable; "all/any" acceptable). NoBroker splits the bachelor filter into **Bachelor Male** + **Bachelor Female**, and exposes each listing's eligibility in the `leaseType`/`leaseTypeNew` fields, so the constraint is applied on the **actual per-listing value**, not inferred:
> - **Kept:** `ANYONE` (46), generic `BACHELOR` = both genders (2), and `COMPANY` whose full eligibility is `ANYONE` (2).
> - **Excluded — single gender (5 listings):** 3 men-only (`BACHELOR_MALE`), 1 women-only (`BACHELOR_FEMALE`), and **1 women-only-bachelor** that reads `FAMILY+BACHELOR_FEMALE` (accepts families + female bachelors but **not** male bachelors → not open to both → excluded). IDs in `findings.md`.

## How the numbers were produced
- `calc_price_per_sqft` = `rent ÷ area` on the captured `area_basis` (**built-up** — NoBroker's basis), ₹/sq.ft/month. Comparable to **Housing.com** (also built-up), **not** to 99acres (carpet).
- `calc_true_monthly_cost` = `rent + extra maintenance + (brokerage + move_in)/12 + deposit×0.005` (STAY_MONTHS=12; deposit at 0.5%/mo opportunity cost). Per `docs/calculations.md`.
- `calc_cost_basis` = **lower-bound for all 50 rows.** Brokerage is a confirmed ₹0 (platform-wide "Without Brokerage") and maintenance is captured, but NoBroker **never exposes move-in/one-time charges**, so one additive cost is always unknown → every figure is a floor, not the full cost (same finding as the NoBroker trial).
- `posted_by` = **owner** for all rows — an **[Inference]**: NoBroker is an owner platform but prints no per-listing owner/broker label logged out (expected, not guaranteed).
- `contact_name` / `contact_phone` left blank — behind the phone+OTP "Get Owner Details" wall.

## Summary (all 50)

| Metric | Value |
|---|---|
| Listings | 50 (all 1 BHK, fully furnished) |
| Property type | Apartment 43 · Independent House 6 · Gated-community villa 1 |
| Tenant preference (as printed) | Anyone 46 · Bachelors 2 · Company 2 (all open to both genders) |
| Rent ₹/month | min 12,000 · median 17,750 · mean 17,080 · max 22,000 |
| ₹/sq.ft/month (built-up) | min 10.0 · median 35.6 · mean 34.1 · max 51.5 |
| Listed by | Owner (inferred) — all rows |
| Brokerage | ₹0 — all rows (NoBroker) |
| Cost completeness | lower-bound — all 50 (move-in never exposed) |

### By searched locality
NoBroker is per-locality; `radius=2.0 km` pulls in adjacent areas, so a search locality's count includes nearby sub-localities (e.g. an HSR search surfaced a Bommanahalli listing). Each row's actual area is in `locality`; the search that surfaced it is noted in `notes`.

| Searched locality | Listings kept | Median rent |
|---|---|---|
| Electronic City | 28 | 16,000 |
| Whitefield | 12 | 18,000 |
| BTM Layout | 5 | 20,000 |
| Koramangala | 2 | 17,000 |
| HSR Layout | 2 | 18,500 |
| Marathahalli | 1 | 20,000 |
| Bellandur | 0 | — |
| Indiranagar | 0 | — |

Electronic City dominates supply for furnished 1BHKs in this budget; Bellandur and Indiranagar returned **zero** matches (genuine — the API returned "0 Rental Homes" for the full filter).

## Best value — lowest true monthly cost (all rows are `lower-bound`)
`tmc` = effective ₹/month incl. amortised one-time costs + deposit opportunity cost. All are floors (move-in unknown).

| # | Locality (listed) | Type | Area (sq.ft) | Rent | Deposit | Maint | ₹/sq.ft | True ₹/mo | Link |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Electronic City (Phase 1) | Apartment | 700 | 12,000 | 24,000 | 0 | 17.1 | 12,120 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronic-city-bangalore-for-rs-12000/8a9fd883831b44cc01831b4b7019009a/detail) |
| 2 | Electronic City | Apartment | 400 | 12,000 | 36,000 | 0 | 30.0 | 12,180 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronic-city-bangalore-for-rs-12000/8a9feb838ae454e9018ae4f0ae7d005a/detail) |
| 3 | Kamasandra (E-City) | Apartment | 500 | 12,000 | 50,000 | 0 | 24.0 | 12,250 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-kamasandra-bangalore-for-rs-12000/8aa9a8b19eab4ea4019eac40697a66bc/detail) |
| 4 | Doddanagamangala (E-City) | Apartment | 600 | 12,000 | 50,000 | 0 | 20.0 | 12,250 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-doddanagamangala-village-bangalore-for-rs-12000/8a9f8fc3937b743901937b7e922f0369/detail) |
| 5 | RG Halli (Whitefield) | Independent House | 300 | 12,000 | 100,000 | 500 | 40.0 | 13,000 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-bengaluru-bangalore-for-rs-12000/8a9ff282860350a001860364a0300bf3/detail) |
| 6 | Electronics Phase 1 (E-City) | Apartment | 500 | 13,000 | 40,000 | 0 | 26.0 | 13,200 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronics-phase-1-electronic-city-bangalore-for-rs-13000/8a9fa88376b70afe0176b7b456a736d8/detail) |
| 7 | Electronic City | Apartment | 400 | 13,000 | 50,000 | 0 | 32.5 | 13,250 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronic-city-bangalore-for-rs-13000/8a9fb28599f1018a0199f154ac7f1a28/detail) |
| 8 | Electronic City | Apartment | 500 | 14,000 | 28,000 | 0 | 28.0 | 14,140 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronic-city-bangalore-for-rs-14000/8a9faf8599ad2d4c0199ad4924a4040e/detail) |
| 9 | Konappana Agrahara (E-City) | Apartment | 500 | 14,000 | 40,000 | 0 | 28.0 | 14,200 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-electronic-city-bangalore-for-rs-14000/8a9f87869619e5f201961a0d7f441029/detail) |
| 10 | Jakkasandra (Koramangala) | Apartment | 500 | 14,000 | 50,000 | 0 | 28.0 | 14,250 | [link](https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-jakkasandra-bangalore-for-rs-14000/ff8081816aa26aa0016aa566083b6ba6/detail) |

## Best value — lowest ₹/sq.ft (built-up; captured as shown, not independently verified)
Favours larger units; very low figures reflect the site's stated built-up area (some "1 BHK" sizes look large — captured verbatim, flagged).

| # | Locality (listed) | Area | Rent | ₹/sq.ft | Note |
|---|---|---|---|---|---|
| 1 | Electronic City | 1,500 | 15,000 | 10.0 | large stated area for a 1 BHK |
| 2 | Electronic City (Sri Sai Balaji villa) | 1,200 | 18,000 | 15.0 | gated-community villa |
| 3 | Horeb Koodarum Church (Whitefield) | 900 | 15,000 | 16.7 | independent house, 2 bath |
| 4 | Electronics City Phase 1 (Avs Stays) | 1,200 | 20,000 | 16.7 | maint ₹1,500 |
| 5 | Electronic City (Phase 1) | 700 | 12,000 | 17.1 | cheapest true cost (above) |
| 6 | Doddanagamangala (E-City) | 600 | 12,000 | 20.0 | |
| 7 | HSR Layout (Sector 7) | 900 | 22,000 | 24.4 | independent house, generic "Bachelor" |

## Caveats (captured as shown, not "corrected" — per the schema/rules)
- **Two likely owner data-entry errors, captured verbatim and flagged in `notes`:** (a) an Electronic City apartment lists **maintenance ₹18,000 = its rent** — its `calc_true_monthly_cost` is inflated and unreliable; (b) a Bommanahalli (HSR search) house lists a **₹5,00,000 deposit** on ₹15,000 rent. Neither appears in the best-value tables.
- **`area_basis` = built-up** for all rows → `calc_price_per_sqft` is comparable to Housing.com, **not** to 99acres (carpet).
- **`property_age`** is NoBroker's age **bucket** (e.g. "Newly Constructed", "1-3 Years", "5-10 Years", "10+ Years"). Mapping verified against detail pages for the "Newly Constructed" (code 0) and "5-10 Years" (code 5) buckets; the intermediate buckets are NoBroker's standard set — treat as the displayed bucket, not an exact age. [Inference]
- **`amenities`** are owner-entered flags read from each listing's amenity map (verified to match the on-page "Amenities" section, incl. one standalone building that over-claims gym/pool); `Water Supply` appended from the Overview. Absent ⇒ blank; this run does not filter on amenities.
- **`maintenance_included`** derived from the displayed "+ ₹N" (extra → false) vs "No Extra Maintenance/Included" (→ true); NoBroker's internal `maintenanceIncluded` API flag is inverted/unreliable and was not used.
- **`contact_phone` / `contact_name`** are phone+OTP-walled on every listing → blank (a gap, not worked around).
- **`posted_by` = owner** is inferred (no printed label); **`brokerage` = 0** by platform design; **`move_in_charges`** never shown → all `calc_true_monthly_cost` are lower bounds.
- **Radius bleed:** `radius=2.0 km` means a searched locality's results include adjacent sub-localities; each row's `notes` records which search surfaced it and its listed locality.
