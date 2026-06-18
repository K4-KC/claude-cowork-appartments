# Recipe: NoBroker (nobroker.in)

> **Cowork — start here.** This recipe is self-contained: to run NoBroker you need only this file. First read **`docs/cowork-run.md`** (the standing run procedure), then follow the site-specific steps below. **Read order:** `docs/cowork-run.md` → this file → `docs/data-schema.md` → `docs/rules.md`. A normal run needs **no kickoff message** — kickoffs are only for first-contact trials of a not-yet-reciped site (`docs/trial-protocol.md`). Note this site is **per-locality** (no city-wide run) — the run must name a locality.

The durable run recipe for NoBroker, distilled from the first-contact trial. If the live site has drifted from what's below, capture the difference in the run's `findings.md` so this recipe can be refreshed.

## Trial status

- **Status:** First-contact trial **complete** (run `data/2026-06-18-trial-nobroker-blr-1bhk/`, 2026-06-18). Raw findings: that run's `findings.md`.
- Sections below are distilled from that trial. Items marked **[Inference]** / **[Unverified]** were seen in a single session — reproduce before trusting.
- **Headline difference vs the other recipes:** NoBroker has **no city-wide search** — the results list silently fails without a locality geo-token, so runs are **per-locality**. Brokerage is genuinely 0 (owner platform); `posted_by` is inferred, not printed.

## URL / entry point

- **Base path:** `https://www.nobroker.in/property/rent/{city}/{LocalityName}` — but the **city/locality path alone is NOT enough**.
- **⚠️ The results list will not render without a locality geo-token.** The results API (`GET /api/v3/multi/property/RENT/filter`) returns HTTP 200 but an app-level failure (`{"status":"fail","status_code":501,"error_message":"No Polygon or point found using token: null"}`) for a city-only/locality-less search; the page sits on **perpetual skeleton placeholders with no error message**. (This is a missing-geo-token, *not* anti-bot or a login wall.) **Fix:** pick a specific locality from the homepage Google autocomplete so a `searchParam` token attaches. **There is no pure city-wide run — iterate per locality.**
- **Deep-linkable? Yes, once the `searchParam` token is in the URL.** Reloading the full filtered URL in a fresh tab reproduced the exact result set (verified).
  - `searchParam` = base64 of `[{"lat":..,"lon":..,"placeId":"<google placeId>","placeName":"<Locality>"}]` — embeds a Google placeId; **cannot be reliably hand-built**. Mint it via the autocomplete once, then **copy and reuse the full URL** (opaque-but-stable, like Housing's token).
  - The **readable filter params CAN be hand-edited** on top of a captured token URL: `type`, `leaseType`, `furnishing`, and especially `rent=min,max` (verified) — the reliable way to set an exact budget.
- **URL template:**
  ```
  https://www.nobroker.in/property/rent/{city}/{Locality}
    ?searchParam={base64 geo-token}   # REQUIRED — without it the list fails (skeletons)
    &radius={km}                      # default 2.0; widens into neighbouring localities
    &city={city}&locality={Locality}
    &type=BHK1[,BHK2,…]               # BHK
    &leaseType=BACHELOR_MALE,BACHELOR_FEMALE
    &furnishing=FULLY_FURNISHED       # FULLY_FURNISHED | SEMI_FURNISHED | NOT_FURNISHED
    &rent={min},{max}                 # raw rupees
    &sharedAccomodation=0             # 0 = whole house (vs PG/shared)
  ```

## Applying the run's filters

| Filter | How to set on NoBroker | Notes |
|---|---|---|
| City | Part of the locality search; `city=bangalore` in URL | City alone is **insufficient** — a locality geo-token is mandatory (see above). |
| Locality | Homepage Google autocomplete → pick locality (mints `searchParam`) | Mandatory. Mind `radius` (default 2.0 km) pulling in neighbouring localities (our Koramangala search surfaced a BTM Layout listing). |
| Listing type (Rent) | "Rent" tab (default); `/property/rent/` path | Clean. |
| BHK | Left rail "BHK Type" → "1 BHK" (`type=BHK1`) | Multi-select. **"1 RK" is a separate button** → pure 1 BHK isolable (unlike 99acres). |
| Budget | "Rent Range" slider **or** URL `rent=min,max` | Slider is non-linear, no numeric input → **set budget via the URL param** (reliable). |
| Furnishing | Left rail "Furnishing" → "Full" (`furnishing=FULLY_FURNISHED`) | Full / Semi / None. |
| Tenant / bachelors | Left rail "Preferred Tenants" → "Bachelor Male" + "Bachelor Female" (`leaseType=BACHELOR_MALE,BACHELOR_FEMALE`) | **Split Male/Female** (no single "Bachelors"). Matching listings often print tenant = "Anyone" (see Gotchas). |

All filters expressible in the UI and encodable in the URL.

## Walking results

- **Infinite scroll** (filter call carries `pageNo`, increments on scroll); no numbered pages or "Load more" seen. Header carries a total count. [Unverified] exact batch size — the in-budget trial set was tiny. "Similar Properties" carousels on detail pages (skip when harvesting).
- **Most schema fields are on the card.** Open each listing's `…/{id}/detail` page only for the ~5 detail-only fields.

### Fastest reliable capture — the results JSON API (added 2026-06-18 run; supersedes DOM/card scraping)

The HTML results list is **unreliable to scrape**: it lazy-renders only ~4 cards, injects non-listing interstitials ("Top Societies in your Search") and a "Premium Properties" promo carousel, and the header count can exceed the rendered cards. **Use the underlying results API instead** — and it returns the "detail-only" fields too, so **per-listing detail-page opens are not needed.**

- Endpoint: `GET https://www.nobroker.in/api/v3/multi/property/RENT/filter` + the results page's own query string (`searchParam` token + filters) + `&pageNo=N`. Call it same-origin from the results page: `fetch(url,{credentials:'include',headers:{accept:'application/json'}})`. Returns `{status:"success", message:"N Rental Homes", data:[…]}`; loop `pageNo=1..` until a short/empty page.
- Each property object carries: `id`, `propertyCode`, `propertyTitle`, `locality`, `city`, `address`/`completeStreetName`, `buildingType` (AP/IH/IF/VL/GC), `type` (BHK), `bathroom`, `propertySize`, `floor`+`totalFloor`, `furnishingDesc`, `propertyAge`, `parkingDesc`, `rent`, `deposit`, `maintenanceAmount`, `leaseType`/`leaseTypeNew`, `availableFrom` (ms epoch), `waterSupply`, `amenitiesMap`, `detailUrl`. [Verified vs 2 detail pages, 2026-06-18.]
- **Field traps (verified):**
  - **`maintenanceIncluded` is inverted/unreliable** — use **`maintenanceAmount`**: >0 ⇒ extra (`maintenance_included=false`); absent/0 ⇒ included (true). Matches the on-page "+ ₹N" vs "No Extra Maintenance".
  - **Amenities:** use **`amenitiesMap`** (per-listing code→bool; matches the page's "Amenities" section). The top-level `amenities` **array (~234 items) is the global catalog — do NOT use it**, and top-level `gym`/`pool`/`lift` booleans are unreliable.
  - **`propertyAge` is a bucket code:** 0→"Newly Constructed", 5→"5-10 Years" [verified]; 1→"1-3 Years", 3→"3-5 Years", 10→"10+ Years" [inferred from NoBroker's standard buckets].
  - **Gender preference IS exposed** (contra the trial's "tenant usually Anyone"): `leaseType`/`leaseTypeNew` give real eligibility — `ANYONE`, `BACHELOR` (=both genders), `BACHELOR_MALE`, `BACHELOR_FEMALE`, `FAMILY`, `COMPANY`, and **combinations** (e.g. `FAMILY+BACHELOR_FEMALE` = women bachelors only). Apply any men-AND-women rule on this, not by inference.
- Tooling note: the Chrome JS-eval tool truncates output ~1 KB and blocks echoed query strings; transfer API rows out by rendering the JSON into the DOM and reading via `get_page_text`.

## Extraction → CSV

Cowork captures only the **captured columns** in `docs/data-schema.md` directly from the page. The derived `calc_` columns are added later by Claude Code in processing — never produce them here.

**Field-coverage map** (`card` = results card · `detail` = detail page only · `login` = login/OTP-walled gap · `absent`):

- **On the card:** `title`, `locality`, `city`, `address`, `rent`, `deposit`, `maintenance` + `maintenance_included` ("+ ₹N" = extra/false; "Included"/"No Extra Maintenance" = true), `area` + `area_basis` (labelled "Builtup" → built-up), `furnishing`, `bhk`, `property_type`, `tenant_preference`, `available_from` ("Possession"), `url`, `listing_id` (hex id in the detail URL).
- **Detail page only:** `bathrooms`, `floor` (site shows "Floor: 3/4"; record in the schema's "X of Y" form, e.g. "3 of 4" — see `docs/data-schema.md`), `property_age` (range bucket "1-3 Years"/"Newly Constructed"), `parking` ("Bike"), `building_name` (only when a named society; blank for "standalone building").
- **Login-walled (gap, leave blank):** `contact_name`, `contact_phone` — behind the "Get Owner Details" phone+OTP modal.
- **Mostly absent:** `amenities` — no dedicated amenities list for standalone/small buildings; only Overview attributes (Water Supply, Gated Security) — capture those. [Unverified] for gated societies. `move_in_charges` — not shown.
- **`brokerage`:** record **0** (platform-wide "Without Brokerage").
- **`posted_by`:** record **owner** as **[Inference]** — no per-listing owner/broker label is printed logged out (only a "Listed by Broker" report-flag exists).

## Gotchas

- **No city-wide run** — the results list fails (perpetual skeletons, silent 501) without a locality geo-token. Iterate per locality; mint the token via the homepage autocomplete.
- **`radius` bleed** — default 2.0 km pulls in neighbouring localities; tighten it or filter by locality in processing.
- **`contact_name` + `contact_phone` are login/OTP-walled** ("Get Owner Details" → phone + OTP). Gaps, left blank — not worked around.
- **Budget slider is coarse/non-linear with no numeric input** — set budget via URL `rent=min,max` instead.
- **`tenant_preference` often prints "Anyone"** even under a Bachelor filter (bachelor-eligible ⊇ Anyone) — don't expect listings to say "Bachelor".
- **`posted_by` is inferred, not printed** (see above); **`brokerage` is 0 by platform design**.
- **"standalone building" is not a society name** → `building_name` blank. Watch truncated text ("Apartmen") — capture as shown (sic).
- Non-blocking first-load tooltips/chat widget ("Search along Metro", Map tooltip, "Natasha" chat) — dismiss; none gate content. No CAPTCHA/rate-limit/forced login from the site.

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| 2026-06-18 | First-contact trial complete: **no city-wide search** (needs locality geo-token; silent skeletons otherwise); deep-linkable with the token; full logged-out browsing except owner contact (phone+OTP); brokerage 0; `posted_by` inferred; near-complete cost set (rent+deposit+maint+brokerage). 3 listings captured + validated; `calc_price_per_sqft` (built-up) added. | Recipe filled. Open: scroll batch size [Unverified]; whether gated-society listings expose a richer amenities list; per-locality run scope noted in `docs/search-config.md`. |
| 2026-06-18 | **Normal run** (`2026-06-18-nobroker-blr-1bhk-furnished-bachelors`, 8 localities, 50 listings). Discovered the **results JSON API** (`/api/v3/multi/property/RENT/filter`) returns all schema fields incl. the detail-only ones (bathroom/floor/age/parking/amenitiesMap) → **detail-page opens unnecessary**. HTML list is unreliable (lazy-render + interstitials). `maintenanceIncluded` flag inverted (use `maintenanceAmount`); `amenities` array is the global catalog (use `amenitiesMap`); `propertyAge` is a bucket code; **gender preference IS exposed** via `leaseType`/`leaseTypeNew`. Bellandur & Indiranagar returned 0 for the filter. | Recipe updated (new "Fast capture via results API" section). Resolves the trial's "tenant usually Anyone" caveat for gender filtering. Open: amenity code→name map has a few [Unverified] labels (SC, CPA). |
