# Findings — Housing.com trial (run `2026-06-18-trial-housing-blr-1bhk`)

**Shared handback doc.** Cowork fills this in while running the trial (`docs/trial-protocol.md`); Claude Code then distills the durable learnings into `docs/sites/housing.md` and validates the captured CSV. This file is the agreed Cowork↔Claude Code communication channel. Keep it factual — record what you actually saw; leave a field blank rather than guessing.

- **Filled by:** Cowork · **Date run:** 2026-06-18 · **Status:** complete

> Caveat on the URL codes below: they were observed in a **single** session and reproduced once in a fresh tab. Treat the specific filter codes as **[Inference]** (expected, not guaranteed). The reliable, verified recipe is "set filters once in the UI, copy the final URL, reuse it" — not hand-editing the opaque codes.

Answer the 7-item checklist from `docs/trial-protocol.md`:

## 1. Logged-out access & walls
Yes — full logged-out browsing works. The homepage already defaulted to Bengaluru; the RENT tab + Search reached the results list, and every result **card and detail page is fully readable without logging in** (rent, area, furnishing, cost breakup, amenities, address, floor, lease type, etc.). No login/app-install modal interrupted browsing at any point during the trial (no forced popup, no scroll-wall, no CAPTCHA, no rate-limiting seen).

The **one** wall is the **contact phone number**: on every detail page it is masked (e.g. `+9198869.....`) behind a "Get Contact Details / Please share your contact" lead form that asks for name + phone. Treated as a **gap** → `contact_phone` left blank for all rows (not worked around).

## 2. Deep-linkable search? (URL template)
Yes — the fully-filtered search is deep-linkable. Reloading the final URL in a **fresh tab reproduced the exact same 255 results with all filter chips active** (1 BHK, ₹12K–₹22K, Fully Furnished, Bachelor). [Verified this session.]

Encoding is an **opaque token scheme in the URL path** (not human-readable `?key=value` query params). A single filter produces a readable slug; **combining** filters switches to a `search-<codes>` form.

```
URL seen (all 5 trial filters applied):
https://housing.com/rent/search-C2G1L4P38f9yfbk7p3m2h1fT99cUgz4

Base (Bengaluru rent flats, no filters):
https://housing.com/rent/flats-for-rent-in-bangalore-karnataka-P38f9yfbk7p3m2h1f

Single-filter readable-slug form (1 BHK only):
https://housing.com/rent/1bhk-flats-for-rent-in-bangalore-karnataka-C2P38f9yfbk7p3m2h1f

URL template (observed codes — [Inference], single session):
https://housing.com/rent/search-<BHK><FURN><LEASE>P38f9yfbk7p3m2h1f<BUDGET_TOKEN>
  P38f9yfbk7p3m2h1f = base search id for "Bengaluru rent flats"
  <BHK>          C2        = 1 BHK   (prefix; confirmed via the 1bhk readable-slug URL)
  <FURN>         G1        = Fully Furnished
  <LEASE>        L4        = Lease type = Bachelor
  <BUDGET_TOKEN> T99cUgz4  = budget ₹12,000–22,000 (suffix after the base id; opaque, decode UNVERIFIED)
```

Caveat: because `<BUDGET_TOKEN>` is opaque I can't confirm hand-editing it to a new range works, nor that codes are stable over time. **Recommended for real runs:** apply filters once in the UI, copy the resulting URL, and reuse it — rather than constructing the token by hand.

## 3. Filter mapping

| Filter | How set on Housing.com | Notes / gaps |
|---|---|---|
| City (Bengaluru) | Homepage city selector / search box; baked into base id `P38f9yfbk7p3m2h1f` (= Bangalore). Homepage already defaulted to Bengaluru. | Path also has readable `...-in-bangalore-karnataka-...`. |
| Listing type (Rent) | "RENT" tab on homepage → `/rent/` path | Clean. |
| BHK (1) | Top filter bar → "BHK Type" dropdown → **1 BHK** (code `C2`) | Options: 1 RK, 1 BHK … 5+ BHK. **Quirk:** got cleared when the budget filter was applied — see §6/§7. |
| Budget (₹12k–22k) | Top filter bar → budget dropdown → **Min/Max numeric inputs** (slider also available); code `T99cUgz4` | Free numeric entry, so any range expressible. |
| Furnishing (Fully) | Top filter bar → "Furnishing" → **Fully Furnished** (code `G1`) | Options: Fully / Semi / Unfurnished. |
| Tenant (Bachelors) | Top filter bar → "More Filters" → **"Lease Type" → Bachelor** (code `L4`) | Site has a single **Bachelor** option (not split male/female) plus Family, Company. Maps cleanly to the brief's "any/bachelors". |

All six trial filters were expressible in the UI; none had to be deferred to processing.

## 4. Field-coverage map
`card` = on the results card · `card(breakup)` = in the card's "see price breakup" popover · `detail` = detail page only · `login` = login/lead-gated (gap) · `absent`.

| Column | Where it lives | Notes |
|---|---|---|
| title | card | e.g. "1 BHK Fully Furnished Flat for rent in Nallurhalli, Whitefield" |
| locality | card | In the title; also on detail breadcrumb/address |
| city | card | Page context (Bengaluru) |
| rent | card | "₹21,000" |
| bhk | card | In title |
| property_type | card | Title: "Flat"=apartment, "Independent Builder Floor"=independent floor |
| area | card | e.g. "610 sq.ft Builtup area" |
| area_basis | card | Card labels it "Builtup area"; detail also gives **carpet area** |
| furnishing | card | "Fully furnished" |
| posted_by | card | Owner listings tagged "Owner"; broker listings show agency name (+ brokerage). All 3 samples were brokers. |
| contact_name | card | Owner/agent name on the card banner (e.g. "Nestora Realty") |
| deposit | card(breakup) | "Refundable deposit"; also on detail as "Security" |
| maintenance | card(breakup) | Also on detail as "Maintenance" |
| maintenance_included | card(breakup) | **[Inference]** — maintenance listed as a separate line ⇒ not included. Expected behavior, not an explicit flag. |
| brokerage | card(breakup) | Also on detail as "Brokerage". 0 for owner listings (not seen in sample). |
| move_in_charges | card(breakup) | "Painting Charges" appears in the popover; **not** on the detail Overview. |
| amenities | card (partial) / detail (full) | Card "Highlights" = curated subset incl. proximity ("Close to X Metro"); detail has full "Amenities" + "Furnishings" lists |
| bathrooms | detail | "Bathrooms: 1" |
| floor | detail | "Floor number: 3 of 5 floors" (floor + total) |
| property_age | detail | "Age of property: 0 year" / "5 years" |
| parking | detail | "1 Covered and 1 Open Parking"; card popover only says parking included/charged |
| available_from | detail | "Available from: Available now" (text token, not a date) |
| tenant_preference | detail | "Lease type: Family / Company / Bachelor"; card sometimes hints "Bachelor Friendly" |
| address | detail | Full "Property Location" line |
| building_name | detail (when present) | Society/project in the address line ("Akshaya Vana", "Nexusnest apt"); **absent** for independent listings with no named society |
| listing_id | detail (URL) | Property ID in the detail URL/title (e.g. `20227168`) |
| contact_phone | **login** (gap) | Masked behind "Get Contact Details" lead form → left blank |
| url | card | Listing link on the card → detail URL |
| source_site / captured_at | bookkeeping | Stamped at capture |

## 5. Pagination model
**Infinite scroll.** First batch ≈ **30 listings** (header reads "Showing 1 – 30 of 255 properties"). Scrolling **auto-appends** more listings; the header count stays **static** (does not update). Recommendation carousels ("Newly added properties in Bengaluru", featured agents) are **injected between batches**, so the list is not one continuous block. No numbered page links and no "load more" button were seen. Implication for a full run: you must scroll repeatedly to materialise all 255 cards, skipping the injected carousels.

## 6. Fastest reliable capture path + bottleneck
Open the **deep-link filtered URL** directly (skip the filter UI). Harvest from the **list cards** everything they carry — title, locality, rent, area + basis, furnishing, bhk, property_type, posted_by, contact_name — plus the **full cost set** via each card's "see price breakup" popover (rent, maintenance, deposit, brokerage, painting/move-in). Then open **each listing's detail page** only for the detail-only fields: bathrooms, floor, property_age, exact parking, available_from, tenant/lease type, full address, building_name, and the complete amenities list. **Bottlenecks:** (1) `contact_phone` is login/lead-gated — an unavoidable gap; (2) ~9 fields require **one detail-page open per listing**, so a full 255-row run is dominated by detail-page visits; (3) infinite scroll + injected carousels make bulk card-loading tedious; (4) the **budget filter silently cleared the BHK filter** mid-setup — always re-check the result count and breadcrumb after each filter (or apply budget first, BHK last).

## 7. Format / unit quirks
- **Currency:** displayed as "₹21,000" (₹ + thousands comma); the price-breakup popover gives clean component amounts. Store as integer rupees.
- **Area:** sq.ft, card-labelled "Builtup area"; detail also lists **carpet area** (e.g. built-up 610 / carpet 580). A "convert unit" toggle (sq.ft ↔ sq.m) exists. Captured `area`=built-up, `area_basis`=built-up; carpet noted in `notes`.
- **available_from:** "Available now" (immediate) — a text token, not a YYYY-MM-DD date. Distinct from "Added X days ago" (the posting date). Processing must normalise.
- **floor:** "3 of 5 floors" encodes both current floor and total.
- **property_age:** "0 year" / "5 years".
- **tenant/lease:** slash-separated "Family / Company / Bachelor"; some listings are "Family / Bachelor" only (no Company).
- **parking:** "1 Covered and 1 Open Parking" / "1 Open Parking" (also a "1 open, 0 closed" line).
- **property_type wording:** "Flat" = apartment; "Independent Builder Floor" = independent floor.
- **"Showing 1–30 of N"** header is static and does not reflect how many cards are actually loaded (infinite scroll).
- **maintenance_included** is not an explicit field — inferred from maintenance being a separate line item. [Inference]
- **Detail URL** embeds the Property ID (= `listing_id`) plus an area/bhk/locality slug.

## Sample captured
**3 listings** written to `housing.csv` (header order preserved, 31 columns each, validated by CSV parse):

| listing_id | locality | rent | deposit | maint | brokerage | type | posted_by |
|---|---|---|---|---|---|---|---|
| 20227168 | S.G. Palya | 21000 | 42000 | 999 | 21000 | apartment | broker (Nestora Realty) |
| 20422355 | Akshayanagar | 18000 | 70000 | 2000 | 18000 | independent floor | broker (Mahendra) |
| 19757273 | Nallurhalli, Whitefield | 22000 | 60000 | 1500 | 22000 | apartment | broker (Bright Keys) |

All three: 1 BHK, Fully Furnished, within ₹12k–22k, `contact_phone` **blank** (login/lead-gated gap), `available_from` = "Available now".

`notes` caveats per row:
- **20227168** — `move_in_charges` = 21000 is the **painting charge** (1 month rent), seen **only** in the card price-breakup popover; carpet area 580 sq.ft.
- **20422355** — carpet area 770 sq.ft; price-breakup popover not opened, so `move_in_charges` left blank (not guessed).
- **19757273** — carpet area 572 sq.ft; lease type **Family/Bachelor only** (no Company); `move_in_charges` left blank for the same reason.

All three sample listings happened to be **broker**-posted (brokerage > 0, "Housing Ultra" agent badge). Owner listings exist in the results (e.g. cards tagged "Owner" with ₹20,000) and would likely show `brokerage` 0 and a different contact pattern — not sampled here; worth confirming in a real run.

---

## Claude Code: validation & calculations (processing pass — 2026-06-18)

**Verdict: capture accepted.** All 3 rows parse; the 31 captured columns are present in schema order; no fabricated values; gaps (`contact_phone`) correctly left blank.

### Validation flags (recorded, not silently fixed)
- **`available_from` holds a text token** ("Available now"), but the schema types it `YYYY-MM-DD`. Convention to settle: keep the token, or normalize "Available now" → the capture date / an `immediate` sentinel. Left as captured for now.
- **`tenant_preference` is an allowed-*set*, not a single value** ("Family/Company/Bachelor"; 19757273 is "Family/Bachelor"). The site lists which tenant types are permitted, slash-joined. Acceptable, but downstream filtering must membership-test, not equality-test.
- **`property_type` = "independent floor"** (20422355) is a variant of the schema's `independent` — normalize on read if we standardize the enum.
- **`floor` includes the word "floors"** ("3 of 5 floors") vs the schema example "3 of 12" — cosmetic; the "X of Y" structure is intact.
- **`move_in_charges` captured inconsistently** — present only where the price-breakup popover was opened (20227168); blank (not guessed) for the other two. Process rule for a real run: **always open the price-breakup popover** so deposit/maintenance/brokerage/move-in are captured uniformly.
- **Sample is 100% broker-posted** — owner `brokerage`/contact pattern unconfirmed; verify in a real run.

### Calculation applied
- **`calc_price_per_sqft`** = `rent ÷ area`, on the captured `area_basis` (built-up for all 3), ₹/sq.ft/month, 2 dp. Defined in `docs/calculations.md`; appended after the captured columns per `docs/data-schema.md`.
- **`calc_true_monthly_cost`** (+ **`calc_cost_basis`**) — later applied per `docs/calculations.md` (`rent + extra maintenance + (brokerage + move-in)/12 + deposit×0.005`). Row 20227168 = `full` (price-breakup popover opened, all cost fields captured); 20422355 and 19757273 = `lower-bound` (`move_in_charges` not captured).

### Per-site comparison table (trial demo — Housing.com, sorted by ₹/sq.ft ascending)
| listing_id | locality | type | area (built-up) | rent | deposit | maint | brokerage | ₹/sq.ft | posted_by |
|---|---|---|---|---|---|---|---|---|---|
| 20422355 | Akshayanagar | independent floor | 780 | 18000 | 70000 | 2000 | 18000 | 23.08 | broker |
| 20227168 | S.G. Palya | apartment | 610 | 21000 | 42000 | 999 | 21000 | 34.43 | broker |
| 19757273 | Whitefield | apartment | 580 | 22000 | 60000 | 1500 | 22000 | 37.93 | broker |

Akshayanagar is the best value per built-up sq.ft; Whitefield the priciest. (Caveat: only 3 rows, all brokers — illustrative, not a real ranking.)

### Calculations still open (need a decision before I commit them)
`calc_true_monthly_cost` is now **defined and applied** (STAY_MONTHS = 12; the *refundable* deposit charged at 0.5%/mo opportunity cost — `docs/calculations.md`). Still open: a commute/distance score (needs target locations) and any weighted ranking — both listed as undecided in `docs/calculations.md`. Flagging, not assuming.
