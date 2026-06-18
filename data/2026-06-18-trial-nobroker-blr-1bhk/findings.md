# Findings — NoBroker trial (run `2026-06-18-trial-nobroker-blr-1bhk`)

**Shared handback doc.** Cowork fills this in while running the trial (`docs/trial-protocol.md`); Claude Code then distills the durable learnings into `docs/sites/nobroker.md` and validates the captured CSV. This file is the agreed Cowork↔Claude Code communication channel. Keep it factual — record what you actually saw; leave a field blank rather than guessing.

- **Filled by:** Cowork · **Date run:** 2026-06-18 · **Status:** complete (3 listings captured + validated)
- **Reference:** `docs/sites/housing.md` and `docs/sites/99acres.md` (same filters, already trialled) — call out where NoBroker differs.
- **Verification convention:** items seen once this session are marked **[Inference]** / **[Unverified]** — reproduce before trusting. The deep-link and the locality-token gate were each re-checked across multiple loads (noted inline).

Answer the 7-item checklist from `docs/trial-protocol.md`:

## 1. Logged-out access & walls

**Logged-out browsing works** for search, filters, the results list, and the **full detail page** — no full-page login wall. Detail pages expose rent, deposit, area, furnishing, floor, bathroom, age, parking, possession, description, photos, posted-on date and view/shortlist counts without an account.

**Biggest gotcha (cost most of the trial to diagnose): the results list will NOT render for a city-only / locality-less search.** NoBroker's results API (`GET /api/v3/multi/property/RENT/filter`) returns **HTTP 200 but an application-level failure** when no locality geo-token is present:
```
{"status":"fail","status_code":501,"error_message":"No Polygon or point found using token: null"}
```
When this happens the page sits on **perpetual skeleton placeholders with no visible error message**. Reproduced across the bare city URL (`/property/rent/bangalore/Bangalore`), an in-page locality search, and "Continue last search". **Fix: you must select a specific locality from the Google-powered autocomplete so a `searchParam` geo-token (base64 lat/lon/placeId) attaches.** Only then do cards render. → **There is no pure city-wide run on NoBroker; runs are per-locality.** (This is the opposite gotcha to the "skeletons = blocked/login" guess — it's a missing-geo-token, not anti-bot or auth.)

**Login-walled fields (gaps):** owner **contact name + phone** are behind the **"Contact" / "Get Owner Details"** button, which opens an **"Enter phone to continue"** modal (mobile number + OTP; URL gets `#signup-login`). Left blank — not worked around.

**Non-blocking interrupts on first load (all dismissable without an account, none gate content):** a "Search along Metro" tooltip (Skip), a Map-feature tooltip (Got it), and a floating "Natasha" chat widget (reappears on navigation).

## 2. Deep-linkable search? (URL template)

**YES — deep-linkable, provided the locality `searchParam` geo-token is in the URL.** Reloading the full filtered URL in a **fresh tab reproduced the exact filtered result set** (verified: header "2 - 1 BHK Fully Furnished … Koramangala", rent label "₹12k to ₹22k", both cards rendered).

```
URL seen (all trial filters, Koramangala, ₹12k–22k):
https://www.nobroker.in/property/rent/bangalore/Koramangala?searchParam=W3sibGF0IjoxMi45MzUxOTI5LCJsb24iOjc3LjYyNDQ4MDY5OTk5OTk5LCJwbGFjZUlkIjoiQ2hJSkxmeVkyRTRVcmpzUlZxNEFqSTd6Z1JZIiwicGxhY2VOYW1lIjoiS29yYW1hbmdhbGEifV0=&radius=2.0&sharedAccomodation=0&city=bangalore&locality=Koramangala&type=BHK1&leaseType=BACHELOR_MALE,BACHELOR_FEMALE&furnishing=FULLY_FURNISHED&rent=12000,22000

The searchParam is base64 of:
[{"lat":12.9351929,"lon":77.62448069999999,"placeId":"ChIJLfyY2E4UrjsRVq4AjI7zgRY","placeName":"Koramangala"}]

URL template:
https://www.nobroker.in/property/rent/{city}/{LocalityName}
  ?searchParam={base64 of [{"lat":..,"lon":..,"placeId":"<google placeId>","placeName":"<Locality>"}]}   # REQUIRED — without it the list fails (skeletons)
  &radius={km}                 # default 2.0; widens to neighbouring localities
  &city={city}                 # e.g. bangalore
  &locality={LocalityName}
  &type=BHK1[,BHK2,...]        # BHK filter
  &leaseType=BACHELOR_MALE,BACHELOR_FEMALE   # tenant filter
  &furnishing=FULLY_FURNISHED  # FULLY_FURNISHED | SEMI_FURNISHED | NOT_FURNISHED(?)
  &rent={min},{max}            # budget in raw rupees
  &sharedAccomodation=0        # 0 = whole house (vs PG/shared)
  &gatedSocietyBucket=control_bucket   # A/B param, not needed
```

**Key caveat / recommended approach:** the `searchParam` token embeds a Google `placeId` + lat/lon and **cannot be reliably hand-built**. Mint it by picking the locality from the homepage Google autocomplete once, then **copy the resulting full URL and reuse it** (treat as opaque-but-stable, like Housing's token). The **readable filter params (`type`, `leaseType`, `furnishing`, `rent`) CAN be hand-edited** on top of a captured token URL — e.g. swapping `rent=12000,22000` is reliable (verified) and is the easy way to set an exact budget (see below).

**vs the other sites:** Housing = deep-linkable opaque path token; 99acres = NOT deep-linkable at all. NoBroker = deep-linkable **but** the essential locality token must be carried in the URL, and there's **no city-wide variant**.

## 3. Filter mapping

| Filter | How set on NoBroker | Notes / gaps |
|---|---|---|
| City (Bengaluru) | Picked as part of the locality search; `city=bangalore` in URL | City alone is **insufficient** — a locality with geo-token is mandatory to get results. |
| Listing type (Rent) | "Rent" tab (default); `/property/rent/` path | Clean. |
| BHK (1) | Left rail "BHK Type" → "1 BHK" (`type=BHK1`) | Multi-select (comma-joined). **"1 RK" is a SEPARATE button** — pure 1 BHK is isolable (unlike 99acres, which bundles 1 RK/1 BHK). |
| Budget (₹12k–22k) | "Rent Range" dual-handle slider (₹0–₹5 Lacs) | Slider is **non-linear (finer at low end), no numeric input → hard to set exactly by drag**. BUT encodes as `rent=min,max` in the URL → **set precisely via the URL param** (verified `rent=12000,22000` → label "₹12k to ₹22k"). Recommend URL over slider. |
| Furnishing (Fully) | Left rail "Furnishing" → "Full" (`furnishing=FULLY_FURNISHED`) | Options: Full / Semi / None. |
| Tenant (Bachelors) | Left rail "Preferred Tenants" → check **"Bachelor Male" + "Bachelor Female"** (`leaseType=BACHELOR_MALE,BACHELOR_FEMALE`) | **Split into Male/Female** (no single "Bachelors") — like 99acres, unlike Housing. Also Family, Company. **Quirk:** matching listings mostly read **"Anyone"** as preferred tenant (bachelor-eligible ⊇ Anyone) — see §7. |

All trial filters expressible in the UI **and** encodable in the URL. Extra filters available (not used): Availability (Immediate / Within 15 / 30 / After 30 days), Property Type (Gated Societies / Apartment / Independent House-Villa / Gated Community Villa), Parking (2/4 Wheeler), "Show Only Lease Properties", and a separate **"Premium Filters"** tab (unexplored).

## 4. Field-coverage map

| Column | card / detail / login / absent | Notes |
|---|---|---|
| title | card (+detail) | e.g. "1 BHK Flat In Standalone Building For Rent In Jakkasandra". Can be truncated in UI. |
| locality | card (+breadcrumb) | Sub-locality shown (Jakkasandra, 80 Feet Road) under the Koramangala parent. |
| city | card | Bangalore. |
| address | card (+detail) | Free-text landmark line, e.g. "Standalone building, 8th Block near Regional Passport Office". |
| rent | card | "₹14,000". |
| maintenance / maintenance_included | card | Shown as "+ ₹N" (extra → `maintenance=N`, `maintenance_included=false`) or "+ Included / No Extra Maintenance" (`maintenance_included=true`). |
| deposit | card | "₹50,000". Sometimes "deposit negotiable" in description. |
| area + area_basis | card | "500 sqft" labelled **"Builtup"** → `area_basis=built-up`. |
| furnishing | card (+detail "Furnishing Status: Full") | Full / Semi / None. |
| bhk | card | 1. |
| property_type | card ("Apartment Type"/"House") + detail | Apartment / Independent House. |
| tenant_preference | card + detail ("Preferred Tenant") | Reads "All"/"Anyone" on these (see §7). |
| available_from | card + detail ("Possession") | "Ready to Move"/"Immediately" token OR a date "Jul 1, 2026". |
| url, listing_id | card (href) | Detail URL `…/{seo-slug}/{ID}/detail`; the hex `{ID}` is the `listing_id` (e.g. `8a9ff58287a722b40187a797cdd247df`). Stable, deep-linkable. |
| bathrooms | **detail** | "Bathroom: 1" in Overview. |
| floor | **detail** | "Floor: 3/4" = floor 3 of 4 total. |
| property_age | **detail** | "Age of Building" range bucket: "1-3 Years", "3-5 Years", "Newly Constructed". |
| parking | **detail** | "Bike" (2-wheeler). |
| building_name | **detail / card title** when a named society ("Gallery Garden Apartmen"); **blank** when "standalone building" | "Standalone building" is a descriptor, not a name → treat as unnamed. |
| contact_name | **login** | Behind "Get Owner Details" phone/OTP modal. Gap, left blank. |
| contact_phone | **login** | Same modal. Gap, left blank. |
| amenities | **largely absent** | **No dedicated amenities list** (no lift/gym/pool/clubhouse section like Housing) for these standalone/small-building listings — only "Overview" attributes (Water Supply, Gated Security, Facing, Pet/Non-Veg Allowed, Balcony). Captured the amenity-relevant ones (Water Supply; Gated Security). **[Unverified]** whether gated-**society** listings expose a richer list — all 3 samples were standalone/small buildings. |
| move_in_charges | absent | Not shown → blank. |
| brokerage | absent → 0 | Site-wide "Without Brokerage" / "Zero Brokerage". Record 0. |

Detail-only extras NOT in schema (ignored or excluded): Facing, Water Supply, Pet Allowed, Non-Veg Allowed, Gated Security, Balcony count, "Posted On" date, view/shortlist/contact counts.

## 5. Pagination model

**Infinite scroll** on the results list — no numbered pages, no "Load more" button observed. A total count is in the header ("11 - 1 BHK Fully Furnished…"; "153" for unfiltered 1 BHK Koramangala) and the underlying filter call carries `pageNo` (increments on scroll). **[Unverified]** exact per-page batch size for NoBroker — the in-budget trial set was tiny (2 in Koramangala proper), so scroll-paging wasn't exercised at volume. "Similar Properties" carousels appear on detail pages (skip when harvesting).

## 6. Fastest reliable capture path + bottleneck

**Fastest path:** (1) From the homepage, pick the **locality** via the Google autocomplete — this mints the `searchParam` geo-token that the results API requires. (2) Apply BHK / furnishing / tenant in the left rail. (3) **Copy the resulting full URL**; if you need an exact budget, edit `rent=min,max` directly (more reliable than the coarse slider). (4) Reuse that URL to reload the filtered list (deep-linkable). (5) **Most schema fields are already on the card** — open each listing's `…/{id}/detail` page only for the ~5 detail-only fields (bathrooms, floor, property_age, parking, exact possession; building_name for named societies).

**Bottlenecks:** (a) **the locality geo-token requirement** — no city-wide run; you must iterate per locality, and mind `radius` (default 2.0 km) pulling in neighbouring localities (our Koramangala search surfaced a **BTM Layout** listing within 2 km). (b) `contact_phone`/`contact_name` login-walled. (c) one detail-page open per listing. (d) coarse rent slider → prefer URL `rent=`.

**Throttling/quirks:** no CAPTCHA, no rate-limit, no forced full-page login from NoBroker during the trial. (One rate-limit we hit was on our own `find` helper/classifier — **our tooling, not the site**.)

## 7. Format / unit quirks

- **Rent slider label** uses "k"/"Lacs" ("₹12k to ₹22k", "₹5 Lacs"); the **URL uses raw rupees** (`rent=12000,22000`). Slider is non-linear.
- **Maintenance:** "+ ₹N" = extra (→ `maintenance=N`, `maintenance_included=false`); "+ Included / No Extra Maintenance" (→ `maintenance_included=true`, `maintenance` blank).
- **area** labelled **"Builtup"** → `area_basis=built-up`; unit sq.ft.
- **floor** "3/4" = floor 3 of 4 total.
- **property_age** is a **range bucket** ("1-3 Years", "3-5 Years") or text ("Newly Constructed") — not a number/year.
- **available_from / Possession:** mixed — a token ("Immediately"/"Ready to Move") OR a date ("Jul 1, 2026" → `2026-07-01`). Normalize in processing.
- **tenant_preference often "Anyone"** even under a Bachelor Male+Female filter (bachelor-eligible includes "Anyone") — so a bachelor filter does not imply the listing prints "Bachelor".
- **building_name** "standalone building" is a descriptor, not a society name → treat as blank/unnamed.
- **Truncated text** in titles/building names ("Apartmen") — capture as shown (sic).

## NoBroker-specific

- **`brokerage` genuinely 0 / absent** — the platform is "Without Brokerage"; the contact modal repeats "Zero Brokerage" and "100% Genuine Owners". Recorded as `0`.
- **`posted_by`:** owner is the **platform default**, but there is **no explicit per-listing "Owner"/"Broker" label** visible logged out. Instead there's a "Report what was not correct → **Listed by Broker**" flag (i.e. owner unless someone flags it). So `posted_by=owner` is an **[Inference]**, not a printed field. No relationship-manager/broker variant appeared in this 3-listing sample.
- **Owner contact without login? NO.** Name + phone are behind the "Get Owner Details" phone/OTP modal → both recorded as gaps (blank).
- **Owner/broker filter or "verified" badge:** no explicit owner/broker results filter seen. Card badges seen: **"Negotiable Rent"**. No "verified" badge in this sample **[Unverified]**. A separate **"Premium Filters"** tab exists (unexplored).

## Sample captured

**3 listings** written to `nobroker.csv` (validated: 31 cols, all rows aligned to header):
1. `…083b6ba6` — 1 BHK, Jakkasandra (Koramangala), **₹14,000**, 500 sqft, Full, maint. Included.
2. `…cdd247df` — 1 BHK, Koramangala (80 Feet Rd), **₹20,000** + ₹1,000 maint., 600 sqft, Full.
3. `…1674f85` — 1 BHK, "Gallery Garden Apartmen", BTM Layout, **₹22,000**, 400 sqft, Full.

**Every row carries a `notes` caveat:** `building_name` blank for the two standalone buildings; `contact_name`/`contact_phone` login-walled; `posted_by=owner` is **[Inference]**; `available_from` mixed token/date; **listing 3 surfaced via the 2 km `radius`** so it's BTM Layout, not Koramangala proper. All within the ₹12k–22k, 1 BHK, fully-furnished, bachelor-eligible target.

---

## Claude Code: validation & calculations (processing pass — 2026-06-18)

**Verdict: capture accepted.** All 3 rows parse; 31 captured columns in schema order; no fabricated values; gaps left blank.

### Validation flags (recorded, not silently fixed)
- **`posted_by=owner` is an [Inference], not a printed field** — NoBroker has no per-listing owner/broker label logged out; "owner" is the platform default. Kept as captured but treat as inferred, not observed.
- **`brokerage=0` site-wide** ("Without Brokerage") — genuine, not a missing value. NoBroker is the first trialled site with a near-complete cost set (rent + deposit + maintenance + brokerage); only `move_in_charges` is absent.
- **Locality bleed:** 8aa9…1674f85 (BTM Layout) surfaced via the default `radius=2.0 km` on a Koramangala search — it's **outside Koramangala proper**. For a locality-scoped run, either tighten `radius` or filter by locality in processing.
- **`available_from` mixed:** token "Immediately" (083b6ba6) vs dates 2026-07-01 / 2026-07-19 (from "Possession") — normalize.
- **`tenant_preference="Anyone"` on all three** despite the Bachelor-Male+Female filter (bachelor-eligible ⊇ Anyone) — so the filter doesn't imply the listing prints "Bachelor".
- **`area_basis=built-up` for all three** → `calc_price_per_sqft` here **is comparable to Housing.com** (also built-up), but **not** to 99acres (carpet).
- **`amenities` sparse** — no dedicated amenities list for these standalone buildings; only Overview attributes captured. [Unverified] whether gated-society listings expose more.
- **`building_name` blank** for the two "standalone building" listings (a descriptor, not a name) — correct.

### Calculation applied
- **`calc_price_per_sqft`** = `rent ÷ area` on the captured `area_basis` (built-up for all 3), ₹/sq.ft/month, 2 dp. Per `docs/calculations.md`.
- **`calc_true_monthly_cost`** (+ **`calc_cost_basis`**) — applied per `docs/calculations.md`. All three rows = `lower-bound`: brokerage is a confirmed 0 and maintenance is captured, but `move_in_charges` is never exposed on NoBroker, so one additive cost stays unknown (real-world-completeness rule — `docs/calculations.md`). _(Updated 2026-06-18 from an earlier `full` flag that used the looser "what the site exposes" reading.)_

### Per-site comparison table (trial demo — NoBroker, sorted by ₹/sq.ft ascending)
| listing_id | locality | area (built-up) | rent | deposit | maint | brokerage | ₹/sq.ft | posted_by |
|---|---|---|---|---|---|---|---|---|
| …083b6ba6 | Jakkasandra (Koramangala) | 500 | 14000 | 50000 | _incl_ | 0 | 28.00 | owner* |
| …cdd247df | Koramangala (80 Feet Rd) | 600 | 20000 | 60000 | 1000 | 0 | 33.33 | owner* |
| …1674f85 | BTM Layout (radius bleed) | 400 | 22000 | 45000 | _incl_ | 0 | 55.00 | owner* |

\* `posted_by` inferred (no printed label). Built-up ₹/sq.ft — comparable to Housing.com, not 99acres.

### Cross-site note (3 of 4 sites in: Housing.com · 99acres · NoBroker)
| | Housing.com | 99acres | NoBroker |
|---|---|---|---|
| Filtered URL deep-linkable | ✅ opaque path token | ❌ filters never reach URL | ✅ but **requires** a locality geo-token; **no city-wide run** |
| Cost fields | rent+deposit+maint+brokerage+move-in (popover) | rent + (sometimes) deposit | rent+deposit+maint+brokerage(=0); no move-in |
| `contact_phone` | login-walled | login-walled | login-walled (phone+OTP) |
| Area basis | built-up | carpet (or Plot Area) | built-up |
| `posted_by` | printed (owner/broker) | printed (owner/dealer) | **inferred** (no label) |
| Run scope | city-wide ok | city-wide ok | **per-locality only** |

**Two cross-cutting consequences for the eventual processing/run model:** (1) `calc_price_per_sqft` is only comparable across sites once `area_basis` is aligned (Housing/NoBroker built-up vs 99acres carpet). (2) **NoBroker can't be run city-wide** — a run that lists localities is required for it; noted in `docs/search-config.md`.
