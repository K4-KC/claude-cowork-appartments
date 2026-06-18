# Recipe: 99acres (99acres.com)

> **Cowork — start here.** This recipe is self-contained: to run 99acres you need only this file. First read **`docs/cowork-run.md`** (the standing run procedure), then follow the site-specific steps below. **Read order:** `docs/cowork-run.md` → this file → `docs/data-schema.md` → `docs/rules.md`. A normal run needs **no kickoff message** — kickoffs are only for first-contact trials of a not-yet-reciped site (`docs/trial-protocol.md`).

The durable run recipe for 99acres, distilled from the first-contact trial. If the live site has drifted from what's below, capture the difference in the run's `findings.md` so this recipe can be refreshed.

## Trial status

- **Status:** First-contact trial **complete** (run `data/2026-06-18-trial-99acres-blr-1bhk/`, 2026-06-18). Raw findings: that run's `findings.md`.
- Sections below are distilled from that trial. Items marked **[Inference]** / **[Unverified]** were seen in a single session — reproduce before trusting.
- **Headline differences vs `docs/sites/housing.md` (same filters):** filtered search is **NOT deep-linkable**; only rent + (sometimes) deposit on cost; a scam interstitial fires per results page; 1 RK and 1 BHK are bundled in one filter.

## URL / entry point

- **Base search URL** (Bengaluru rent residential): `https://www.99acres.com/search/property/rent/bangalore?city=20&preference=R&res_com=R` (`city=20` = Bengaluru, `preference=R` = Rent, `res_com=R` = Residential).
- **Deep-linkable? No.** Applying BHK / budget / furnishing / tenant via the left rail does **not** change the address bar — only `city`, `preference`, `res_com` persist. Reloading the URL in a fresh tab reproduces only the **base** (unfiltered) set, not the filtered results. Filters go to a private `…/api-aggregator/srp/search` AJAX call (per `docs/rules.md`, **not** our path — don't construct URLs from it). **Implication: filters must be re-applied in the UI every session; there is no shareable filtered link.**
- **Listing detail pages ARE deep-linkable & stable:** `https://www.99acres.com/<seo-slug>-spid-<ID>`; the `spid` is the `listing_id` (e.g. `spid-K91920752` → `K91920752`).
- Full logged-out browsing works (search, filters, results, detail pages) — see Gotchas for the walls.

## Applying the run's filters

| Filter | How to set on 99acres | Notes |
|---|---|---|
| City | Search box → type "Bangalore" → pick "Bangalore (City)" (`city=20`) | Sub-city options (West/East/North/Central/South) also offered. |
| Listing type (Rent) | "Rent" tab on the search bar → `preference=R`, `/rent/` path | Clean. |
| BHK | Left rail "No. of Bedrooms" → "1 RK/1 BHK" (`bedroom_num=1`) | **QUIRK: 1 RK and 1 BHK share one button** — can't isolate pure 1 BHK; results include 1 RK studios. Isolate in processing if needed. |
| Budget | Left rail "Budget" → Min & Max dropdowns (₹1,000 steps) | Free range. Sent internally as bucket IDs (e.g. 209/219), not raw rupees [Inference] — part of why the URL can't be hand-edited. |
| Furnishing | Left rail "Furnishing status" → "Furnished" (`furnish=1`) | Options: Semifurnished / Furnished / Unfurnished. "Furnished" = fully furnished. |
| Tenant / bachelors | Left rail "Available for" → select both "Single Men" + "Single Women" (`tenant_pref=SW,SM`) | **QUIRK: no single "Bachelors" option** (split into Single Men / Single Women / Family / Company Lease). **UI QUIRK: the list re-orders selected items to the top as you click**, which can cause mis-clicks — verify each toggle. Detail pages show a combined "Bachelors (Men/Women)" label. |

All filters expressible (with the quirks above); only isolating *pure* 1 BHK from the 1 RK/1 BHK bundle must be deferred to processing.

## Walking results

- **Infinite scroll.** First batch = 25 cards (`page_size=25`); scrolling auto-fires `page=2,3,…` and appends more. **No "Load more"/"Next" button.** The "N results" header is a static total; "Similar properties" carousels are injected between batches (skip them).
- **Open a listing's `-spid-` detail URL** only for the detail-only fields (below). A full run is dominated by one detail-page open per listing.

## Extraction → CSV

Cowork captures only the **captured columns** in `docs/data-schema.md` directly from the page. The derived `calc_` columns are added later by Claude Code in processing — never produce them here.

**Field-coverage map** (`card` = results card · `detail` = detail page only · `login` = login/lead-walled gap · `absent` = site doesn't expose it):

- **On the card (and detail):** `title`, `locality`, `city`, `rent`, `bhk`, `property_type`, `area` + `area_basis`, `furnishing`, `bathrooms`, `posted_by` (Owner/Dealer), `building_name` (society/project/landmark = card's bold top line), `deposit` (a ₹ figure **or** an "N months rent" rule — and sometimes absent), `listing_id` (the `spid`), `url`.
- **Detail page only:** full `address`, `floor` ("1st of 3 Floors"), `property_age` (range bucket "1 to 5 Year Old"), `parking`, full `amenities`, `available_from` ("Immediate"/"Ready to move"/a date), `tenant_preference` ("All"/"Bachelors (Men/Women)"/"Family"), `contact_name`.
- **Login-walled (gap, leave blank):** `contact_phone` — masked behind a "View Advertiser Details" lead form.
- **Not provided by this site (leave blank):** `maintenance`, `maintenance_included`, `brokerage`, `move_in_charges`. ⚠️ Real coverage gap vs Housing.com — 99acres exposes essentially **only rent and (sometimes) deposit** for cost.

## Gotchas

- **Filtered search is not in the URL** (see above) — re-apply filters in the UI each session; no shareable filtered link.
- **`contact_phone` is login/lead-walled** (masked `+91 98***543**` behind a name+phone form). Gap, left blank — not worked around. Owner/dealer **name** is visible without the wall.
- **Scam-warning interstitial** ("Never pay booking amount…") fires on **every fresh results-page load** (incl. reloads). Dismiss with "Ok, understood"; no account needed; absent on detail pages.
- **"Newest first" sort is login-gated** ("Login to Unlock"); Relevance / Price-Low-High / Price-High-Low work logged out.
- **1 RK/1 BHK bundled** → studios appear in "1 BHK" results; `bhk` ambiguous for studios.
- **`area_basis` varies** — Carpet / Super Built-up / Built-up / **Plot Area**. "Plot Area" is not living area and can dwarf the unit (sample: Plot 1200 vs Carpet 600 sq.ft) — capture the living-area basis and flag.
- **Poster-entered fields can be wrong** — one listing's `available_from` ("30 November 2030") contradicted its "Ready to move"; treat dates skeptically.
- **Tenant filter re-orders on click** (UI quirk) — verify each toggle after selecting.
- No CAPTCHA, no rate-limiting, no forced full-page login seen during the trial.

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| 2026-06-18 | First-contact trial complete: full logged-out browsing; filtered search **not** deep-linkable; scam interstitial per SRP; phone login-walled; 1 RK/1 BHK bundled; only rent + (sometimes) deposit on cost. 3 listings captured + validated; `calc_price_per_sqft` (carpet) added. | Recipe filled. Open: whether maintenance/brokerage ever appear on some listings [Unverified]; whether the unfiltered SRP can be filtered via on-page sort+scan more cheaply than re-applying the rail each session. |
