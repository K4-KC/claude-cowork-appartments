# Findings — 99acres trial (run `2026-06-18-trial-99acres-blr-1bhk`)

**Shared handback doc.** Cowork fills this in while running the trial (`docs/trial-protocol.md`); Claude Code then distills the durable learnings into `docs/sites/99acres.md` and validates the captured CSV. This file is the agreed Cowork↔Claude Code communication channel. Keep it factual — record what you actually saw; leave a field blank rather than guessing.

- **Filled by:** Cowork · **Date run:** 2026-06-18 · **Status:** complete
- **Reference:** `docs/sites/housing.md` (same filters, already trialled) — call out where 99acres differs.

> Notes on confidence: items seen in a single session are tagged **[Inference]** (a reasonable read of observed behaviour, not guaranteed) or **[Unverified]** (couldn't confirm). Reproduce before trusting blindly.

Answer the 7-item checklist from `docs/trial-protocol.md`:

## 1. Logged-out access & walls
**Logged-out browsing works fully** — search, all left-rail filters, results, and listing detail pages render without an account.

Walls / interruptions seen:
- **Scam-warning interstitial** — a modal "Never pay booking amount without visiting the property" (SCAMMERS WILL ASK FOR: property visit charges / gate pass booking / refundable booking amount) fires on **every fresh load of a search-results (SRP) page** (fired on the initial search and again when I reloaded the SRP URL in a new tab). Dismissed with **"Ok, understood"**; no account needed. It does **not** appear on listing detail pages.
- **Phone is login/lead-walled** — clicking "Contact Owner"/"Contact Dealer" opens a modal that shows a **masked** number (e.g. `+91 98***543**`) and requires submitting *your own* name + phone (a lead form, "View Advertiser Details") to unmask it. → `contact_phone` recorded as a **GAP / left blank**, not worked around. The owner/dealer **name** is visible without the wall.
- **"Newest first" sort is gated** — that one sort option is labelled "Login to Unlock". Relevance / Price-Low-High / Price-High-Low all work logged out.
- No CAPTCHA, no rate-limiting, and no forced full-page login seen during the trial. Login/Register prompts exist but are non-blocking.

Difference vs Housing.com: Housing had only the phone gated and **no** forced interstitial; 99acres adds the per-SRP scam modal and a login-gated sort.

## 2. Deep-linkable search? (URL template)
**No — the filtered search is NOT deep-linkable.** Applying all five filters via the UI left the address bar at the base city/rent URL; only `city`, `preference`, `res_com` persist. BHK, budget, furnishing and tenant are applied via an internal AJAX call and are **lost on reload/share**.

- Verified: reloading the address-bar URL in a fresh tab reproduced only the **base** set (city + rent + residential → 27,130+ results, all BHKs/budgets), **not** the filtered 494 (1 BHK / ₹12–22k / furnished / bachelors). The page `rel=canonical` is also the unfiltered base `/property-for-rent-in-bangalore-ffid`.
- The filters go to a **private API** `…/api-aggregator/srp/search` (params incl. `bedroom_num=1`, `furnish=1`, `tenant_pref=SW,SM`, `budget_min=209&budget_max=219` [bucket IDs, not raw rupees — [Inference]], `page_size=25`, `page=N`). From page 2 on, the filter set is serialized into an opaque base64 `encrypted_input` token (decodes to a pipe/hash blob `R | QS | R |#6# | 1 … price_a … 209 | 219 … 20 … SW,SM …`) plus an `exclusionPropIds` dedup list. Per `docs/rules.md` this hidden API is **not** our fast path — don't build URLs from it.

```
URL seen (after applying ALL 5 filters; only the base survives):
  https://www.99acres.com/search/property/rent/bangalore?city=20&preference=R&res_com=R

URL template (only the reproducible part — city + rent + residential):
  https://www.99acres.com/search/property/rent/bangalore?city=<CITY_ID>&preference=R&res_com=R
    city=20  -> Bengaluru ("Bangalore")
    preference=R -> Rent
    res_com=R    -> Residential
  BHK / budget / furnishing / tenant: NOT in the URL — must be re-applied in the UI each session.

Listing DETAIL pages ARE deep-linkable & stable (slug + spid; spid == listing_id):
  https://www.99acres.com/<seo-slug>-spid-<ID>
  e.g. .../1-bhk-bedroom-apartment-flat-for-rent-in-bangalore-east-650-sqft-spid-K91920752
```

Difference vs Housing.com: Housing's fully-filtered URL **was** deep-linkable (opaque token in the path, but reload reproduced the result set). On 99acres it is **not** — the filtered state never reaches the URL.

## 3. Filter mapping

| Filter | How set on 99acres | Notes / gaps |
|---|---|---|
| City (Bengaluru) | Search box → type "Bangalore" → pick "Bangalore" (City). Internally `city=20`. | Sub-city options also offered (Bangalore West/East/North/Central/South). |
| Listing type (Rent) | "Rent" tab on the search bar → `preference=R`, path `/rent/`. | Clean. |
| BHK (1) | Left rail "No. of Bedrooms" → **"1 RK/1 BHK"** button (`bedroom_num=1`). | **QUIRK: 1 RK and 1 BHK are bundled in one button** — can't isolate a pure 1 BHK from 1 RK/studio here. Results include 1 RK studios. Isolate in processing if needed. |
| Budget (₹12k–22k) | Left rail "Budget" → Min & Max dropdowns, ₹1,000 steps. Selected 12,000 / 22,000. | Free range, any value selectable. Sent internally as **bucket IDs** (209/219), not raw rupees [Inference] — another reason the URL can't be hand-edited. |
| Furnishing (Fully) | Left rail "Furnishing status" → **"Furnished"** (`furnish=1`). | Options: Semifurnished / Furnished / Unfurnished. "Furnished" = fully furnished. |
| Tenant (Bachelors M&F) | Left rail "Available for" → select **both "Single Men" + "Single Women"** (`tenant_pref=SW,SM`). | **QUIRK: no single "Bachelors" option** — split into Single Men / Single Women / Family / Tenants with Company Lease. (Detail pages *do* show a combined "Bachelors (Men/Women)" label.) **UI QUIRK: the "Available for" list re-orders selected items to the top as you click**, which caused a mis-click (Family got selected) — verify each toggle after clicking. |

All filters are expressible (with the two quirks above). None had to be deferred to processing, except isolating *pure* 1 BHK from the 1 RK/1 BHK bundle.

## 4. Field-coverage map
`card` = results card · `detail` = listing detail page only · `login` = login/lead-walled (a gap) · `absent` = site doesn't expose it.

| Column | card / detail / login / absent | Notes |
|---|---|---|
| title | card + detail | |
| locality | card + detail | |
| city | card + detail | |
| rent | card + detail | "₹12,000 /month". |
| bhk | card + detail | 1 RK shows as "1 BHK" on the card (see quirks). |
| area | card + detail | sqft, with sq.m. in parentheses on detail. |
| area_basis | card + detail | Labelled on both: **Carpet / Super Built-up / Built-up / Plot Area** (varies per listing). |
| furnishing | card + detail | Card shows a "FURNISHED" ribbon. |
| deposit | card + detail | Card: "+ Deposit ₹X" **or** "+ Deposit N months rent". Detail: "View Rent Details" popover + a "Security Deposit" line. **May be ABSENT** (1 of 3 sampled had none). |
| maintenance | mostly **absent** | The detail "View Rent Details" popover showed only **Rent + Deposit** on all sampled listings — no maintenance line. (≠ Housing, whose popover carried maintenance/brokerage/painting.) Occasionally present in detail [Unverified]. |
| maintenance_included | absent | Not stated. ("Electricity & Water Charges: included / not included" appears on detail but is *utilities*, not maintenance.) |
| brokerage | absent | No explicit brokerage figure on owner **or** dealer listings sampled. |
| move_in_charges | absent | Not shown. |
| amenities | detail only | Full list on detail ("Why consider…", "Facilities", "Furnishing Details"). Card shows 1–2 tags + "+N". |
| floor | detail (reliable) | "1st of 3 Floors" / "GF out of 1 Floors". Sometimes also a card tag ("2nd out of 4 Floors") but inconsistent. |
| parking | detail only | e.g. "1 Covered, 2 Open"; absent on some. |
| property_age | detail only | Range bucket: "1 to 5 Year Old", "5 to 10 Year Old" (not an exact year). |
| available_from | detail only | "Immediate" / "Ready to move" / a date. |
| tenant_preference | detail only | "Available For: All / Bachelors (Men/Women) / Family". **Not on the card.** |
| posted_by | card + detail | Owner / Dealer. |
| contact_name | detail only | Owner/Dealer Details section (e.g. "Rs Rao", "Gopi Nath"). Card may show an agency/company name. |
| contact_phone | **login** | Masked + lead form → **GAP, left blank.** |
| building_name | card + detail | Society/project/landmark name = the card's bold top line (e.g. "gopi homes", "godrej air nxt", "Basavagangotri Layout"). Present for most; may be a layout/landmark rather than a true society. |
| address | detail only | Full address w/ nearest landmark in "About Property". Card shows locality only. |
| bathrooms | card + detail | Card shows "1 Bath". |
| listing_id | card + detail | The **`spid`** in the detail URL (e.g. `spid-K91920752` → `K91920752`). |

## 5. Pagination model
**Infinite scroll.** First batch = **25 cards** (`page_size=25`); scrolling auto-fires `page=2`, `page=3`, `page=4` … to `…/srp/search` and appends more (74 unique cards after a few scrolls). **No "Load more"/"Next" button.** "Similar properties" carousels are **injected between batches** (skip them when harvesting). The result header ("494 results | …") is a static total. (A numbered-page footer may also exist, but auto-load is the operative model.) Page 2+ requests carry an `exclusionPropIds` list so already-shown listings aren't repeated.

## 6. Fastest reliable capture path + bottleneck
Open the base city/rent URL → dismiss the scam interstitial once → apply the 5 filters in the left rail (**set Budget first, then re-verify; watch the re-ordering "Available for" list**) → scroll to lazy-load cards. **Most comparison fields come straight off the cards** (rent, deposit, bhk, bathrooms, area + area_basis, furnishing, posted_by, locality, city, property_type, building_name, title, the spid/listing URL). **Open each listing's `-spid-` detail URL only for the detail-only fields** (full address, floor, property_age, parking, full amenities, available_from, tenant_preference, contact_name). **Main bottleneck:** one detail-page open per listing for ~8 detail-only fields (notably `tenant_preference` + `floor` + `amenities` + `contact_name`), and `contact_phone` is permanently login-walled. **Secondary friction:** the scam interstitial on every SRP load; the re-ordering tenant filter; and — because BHK/budget/furnishing/tenant never reach the URL — filters must be re-applied by hand each session (no shareable filtered link, unlike Housing).

## 7. Format / unit quirks
- **1 RK/1 BHK bundling** — 1 RK studios appear in "1 BHK" results; a 1 RK's `bhk` is ambiguous (logged `bhk=1` + a note). Filter true 1 BHK in processing if required.
- **area_basis varies** — Carpet / Super Built-up / Built-up / **Plot Area**. "Plot Area" is not a standard living-area basis and can be far larger than the unit (sample 1: Plot 1200 vs Carpet 600 sqft); some detail pages list **both**. Captured Carpet where available — normalize/flag basis before computing ₹/sqft.
- **Area string** — "600 sqft (55.74 sq.m.)"; strip the parenthetical sq.m.
- **Deposit** — sometimes a number ("₹60,000"), sometimes a rule ("Deposit 3 months rent" / "2 months rent"), sometimes **absent**. Normalize to ₹ using rent where it's a multiple.
- **available_from** — a text token ("Immediate" / "Ready to move") **or** a date. One sampled listing showed an implausible far-future date ("30 November 2030") that contradicts its own "Ready to move" → treat poster-entered dates skeptically.
- **rent** — "₹12,000 /month"; clean integer after stripping ₹ and comma.
- **floor** — "1st of 3 Floors", "GF out of 1 Floors" (ground floor) — parse floor + total.
- **property_age** — range bucket ("1 to 5 Year Old"), not an exact year.
- **tenant label mismatch** — filter uses "Single Men"/"Single Women"; detail page shows "Bachelors (Men/Women)" / "All".
- **Budget** sent as internal bucket IDs (209/219) not rupees [Inference] — doesn't affect captured data, but means the URL can't be hand-edited to change budget.

## Sample captured
**3 listings** written to `99acres.csv` (all matched the filters; all 1 BHK @ ₹12,000; price-ascending sort):

| spid (listing_id) | type | posted_by | area | deposit | notes flag |
|---|---|---|---|---|---|
| F82539712 | Builder Floor, Kumbalgodu | Owner (Rs Rao) | 600 carpet | ₹60,000 | Plot area 1200 also listed (card's primary); captured carpet 600. |
| C90865920 | 1 RK Studio, Pragathi Layout | Dealer (Gopi Nath) | 354 carpet | ₹36,000 | 1 RK studio, **not a true 1 BHK** — bundled-filter artifact. tenant = "Bachelors (Men/Women)". |
| K91920752 | Apartment, A.S. Raju Nagar | Owner (Raju) | 650 carpet | *(blank)* | `available_from` "30 November 2030" contradicts "Ready to move" (poster error); **no deposit shown**; photos not shared. |

`contact_phone` blank on all three (login wall). CSV validated: 31 columns per row, matches the schema header order.

---

## Claude Code: validation & calculations (processing pass — 2026-06-18)

**Verdict: capture accepted.** All 3 rows parse; 31 captured columns in schema order; no fabricated values; gaps left blank.

### Validation flags (recorded, not silently fixed)
- **`available_from` = `2030-11-30` (K91920752) is a poster error** — it contradicts the listing's own "Ready to move". Kept as captured (source-of-truth), flagged in the row's `notes`; any commute/availability logic must treat far-future dates skeptically. The other two are the text token "Immediate".
- **`bhk=1` is ambiguous for C90865920** — it's a **1 RK studio**, surfaced by 99acres' bundled "1 RK/1 BHK" filter, not a true 1 BHK. Flagged in `notes`; isolate pure 1 BHK in processing if a run requires it.
- **`area_basis` = carpet for all 3** (Housing.com's trial was built-up). So `calc_price_per_sqft` here is **carpet-basis** and **not directly comparable** to Housing's built-up ₹/sq.ft — align the basis before any cross-site comparison.
- **`property_type` variants** — "Builder Floor" (≈ independent), "Studio Apartment (1 RK)", "Apartment". Normalize on read if we standardize the enum.
- **Site does not expose `maintenance`, `maintenance_included`, `brokerage`, `move_in_charges`** — correctly blank (absent), not missed. `deposit` is also absent on one owner listing (K91920752). This is a real coverage gap vs Housing.com, whose popover carried the full cost set.
- **`tenant_preference`** captured from the detail page as "All" / "Bachelors (Men/Women)" — note the filter UI uses different labels ("Single Men"/"Single Women").

### Calculation applied
- **`calc_price_per_sqft`** = `rent ÷ area` on the captured `area_basis` (carpet for all 3), ₹/sq.ft/month, 2 dp. Per `docs/calculations.md`.

### Per-site comparison table (trial demo — 99acres, sorted by ₹/sq.ft ascending)
| listing_id | locality | type | area (carpet) | rent | deposit | maint | posted_by | ₹/sq.ft |
|---|---|---|---|---|---|---|---|---|
| K91920752 | A.S. Raju Nagar | apartment | 650 | 12000 | _(none)_ | _(n/a)_ | owner | 18.46 |
| F82539712 | Kumbalgodu | builder floor | 600 | 12000 | 60000 | _(n/a)_ | owner | 20.00 |
| C90865920 | Pragathi Layout | 1 RK studio | 354 | 12000 | 36000 | _(n/a)_ | dealer | 33.90 |

Caveats: 3 rows only; C90865920 is a 1 RK studio (not a true 1 BHK); K91920752 has no deposit shown and a bogus availability date. ₹/sq.ft is **carpet-basis** — do not compare against Housing.com's built-up figures.

### Cross-site note (Housing.com vs 99acres, same filters)
- **Deep-linkability:** Housing = yes (opaque token URL reproduces the filtered set); 99acres = **no** (BHK/budget/furnishing/tenant never reach the URL — only city/rent/residential survive). On 99acres, filters must be re-applied in the UI each session.
- **Cost coverage:** Housing exposed deposit + maintenance + brokerage + move-in (via a card popover); 99acres exposes **only rent and (sometimes) deposit**.
- **Walls:** both gate `contact_phone`; 99acres adds a per-SRP scam interstitial and a login-gated "Newest first" sort.
- **Area basis:** Housing built-up; 99acres carpet (and sometimes "Plot Area", which is not living area).
