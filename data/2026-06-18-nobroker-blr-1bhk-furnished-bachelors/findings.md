# Findings — NoBroker normal run (`2026-06-18-nobroker-blr-1bhk-furnished-bachelors`)

**Shared handback doc** (Cowork ↔ Claude Code). Filled by Cowork during the run; durable learnings should be distilled into `docs/sites/nobroker.md`. Items seen this session and not yet independently reproduced are tagged **[Inference]/[Unverified]**.

- **Driver:** Cowork (Claude in Chrome). **Date:** 2026-06-18. **Status:** complete — 50 listings captured + processed.
- **Filters:** Bengaluru · Rent · 1 BHK · ₹12,000–22,000 · Fully Furnished · bachelor-eligible, kept only both-gender / any.
- **Localities (per-locality, no city-wide):** BTM Layout, Koramangala, HSR Layout, Marathahalli, Whitefield, Bellandur, Electronic City, Indiranagar.

## What matched the recipe
- **No city-wide search** — confirmed. Each locality needed its own `searchParam` geo-token, minted from the homepage Google autocomplete (first suggestion = the locality in every case). Deep-link/token behaviour as documented.
- **Filters via URL** — `type=BHK1`, `furnishing=FULLY_FURNISHED`, `leaseType=BACHELOR_MALE,BACHELOR_FEMALE`, `rent=12000,22000` all applied correctly on top of the token URL.
- **`brokerage` = 0** platform-wide; **`contact_name`/`contact_phone`** phone+OTP-walled (blank); **`posted_by`** has no printed label (owner = [Inference]); **`move_in_charges`** never shown; **`area_basis`** = built-up.

## New / divergent findings (please distil into `docs/sites/nobroker.md`)

1. **The results JSON API is the fastest, most reliable capture path — and it carries the detail-only fields, so per-listing detail-page opens are NOT needed.**
   - `GET https://www.nobroker.in/api/v3/multi/property/RENT/filter` + the results page's own query string (`searchParam` token + filters) + `&pageNo=N`. Returns `{status:"success", message:"N Rental Homes", data:[…]}`. Same-origin `fetch` with `credentials:'include'` from the results page.
   - Each property object includes everything the schema needs **including the trial's "detail-only" fields**: `bathroom`, `floor`+`totalFloor`, `propertyAge`, `parkingDesc`, plus `rent`, `deposit`, `maintenanceAmount`, `propertySize`, `furnishingDesc`, `leaseType`/`leaseTypeNew`, `availableFrom` (ms epoch), `buildingType`, `waterSupply`, `amenitiesMap`, `propertyTitle`, `address`, `detailUrl`, `propertyCode`. → **Detail-page opens were unnecessary this run.** [Verified against 2 detail pages.]
   - Pagination: loop `pageNo=1..` until a short/empty page. Result sets here were small (≤33/locality).

2. **The HTML results list is unreliable to scrape.** It lazy-renders only ~4 cards, then injects non-listing interstitials ("Top Societies in your Search") and a "Premium Properties" promo carousel; the header count ("6 Rental Homes") can exceed the rendered cards. DOM-based card harvesting under/over-counts. Use the API (point 1).

3. **Per-listing gender preference IS exposed** (contradicts the trial's "tenant usually prints Anyone"). `leaseType` / `leaseTypeNew` give the real eligibility: `ANYONE`, `BACHELOR` (gender-neutral = both), `BACHELOR_MALE`, `BACHELOR_FEMALE`, `FAMILY`, `COMPANY`, and **combinations** (e.g. `FAMILY+BACHELOR_FEMALE`). The on-card "Preferred Tenants" also prints "Male"/"Female"/"All". → the men-AND-women rule was applied on data, not inferred. **Edge case:** `FAMILY+BACHELOR_FEMALE` accepts families + female bachelors but **not** male bachelors → women-only for bachelors → excluded.

4. **`maintenanceIncluded` API flag is inverted/unreliable** — it read `true` for listings displaying "+ ₹N extra" and `false` for "No Extra Maintenance". Use **`maintenanceAmount`**: >0 ⇒ extra (`maintenance_included=false`); absent/0 ⇒ included (`maintenance_included=true`). Matches the on-page "+ ₹N" vs "Included" display. [Verified on a detail page.]

5. **Amenities:** the real per-listing list is **`amenitiesMap`** (code→bool, e.g. `RWH/HK/STP/PB/SECURITY/VP/SERVANT`), which matched a detail page's "Amenities" section exactly (incl. a standalone building that over-claims gym/pool). The top-level **`amenities` array (~234 items) is the global catalog — do NOT use it**, and the top-level `gym`/`pool`/`lift` booleans are unreliable. Code→name map used: INTERCOM, AC, RWH, HK, INTERNET, LIFT, CLUB, GP, FS, STP, PARK, SC, PB, CPA, SECURITY, POOL, GYM, VP, SERVANT (PB=Power backup, VP=Visitor parking, SC=Shopping centre, CPA=Children's play area — last few **[Unverified]** display names).

6. **`propertyAge` is a bucket code, not exact years.** 0 → "Newly Constructed", 5 → "5-10 Years" (both [Verified] vs detail pages); 1→"1-3 Years", 3→"3-5 Years", 10→"10+ Years" mapped from NoBroker's standard buckets **[Inference]**.

7. **Zero-result localities:** Bellandur and Indiranagar returned `0 Rental Homes` for the full filter (genuine API zero, token minted fine). Electronic City (28 kept) and Whitefield (12) dominate supply.

8. **Tooling note (not a site issue):** the agent's JS-eval tool truncates output at ~1 KB and blocks any echoed query string; the API rows were transferred out by rendering the JSON into the DOM and reading it via `get_page_text`. No CAPTCHA, throttle, or login wall from NoBroker this run.

## Gender exclusions (5 single-gender listings, for transparency)
Single-gender bachelor listings dropped (not in `nobroker.csv`):
- **Men-only (`BACHELOR_MALE`), 3:** `…5374195f` (BTM Layout 1st Stage), `…882fae`, `…b018d4` (both Electronic City).
- **Women-only, 2:** `…f40d17` (`BACHELOR_FEMALE`, Electronic City) and `…15323223` (`FAMILY+BACHELOR_FEMALE`, Electronic City — families + women bachelors only).
(4 dropped at capture by the single-male/female rule; the `FAMILY+BACHELOR_FEMALE` one dropped in processing by the precise leaseTypeNew rule. Separately, 1 internal Electronic-City duplicate `…4c17036a` was removed in de-dup. Net: 56 fetched − 5 single-gender − 1 duplicate = 50.)

## Data quirks captured verbatim and flagged (not corrected)
- One Electronic City apartment lists **maintenance ₹18,000 (= its rent)** → `calc_true_monthly_cost` inflated; flagged in `notes`.
- One Bommanahalli (HSR search) house lists a **₹5,00,000 deposit** on ₹15,000 rent → likely error; flagged.
- Some "1 BHK" stated **built-up areas are large** (1,200–1,500 sq.ft) → very low ₹/sq.ft; captured as shown.

## Output
- `nobroker.csv` — 50 rows, 31 captured + 3 derived columns (`calc_price_per_sqft`, `calc_true_monthly_cost`, `calc_cost_basis`). All `calc_cost_basis = lower-bound` (move-in never exposed).
- `comparison.md` — per-site summary, by-locality, best-value shortlists.
