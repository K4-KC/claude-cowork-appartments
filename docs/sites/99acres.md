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

- **Numbered pager, ~25 cards/page, windowed/virtualised.** (Updated 2026-06-18 — supersedes the earlier "infinite scroll, no Next button" note.) The SRP has a **numbered footer pager** (e.g. *"Page N of 20"*, ~25 listings/page) with page-number anchors (`<a>`; current = `<a class="Pagination__active">`) and a **"Next Page >"** link (`<a class="list_header_bold">`, shown once you reach the end of the visible 1–10 window). The list is **virtualised: only ~25 cards live in the DOM at once**, and a card's `-spid-` anchor (its `listing_id`/`url`) attaches **only after that card's photo lazy-loads**. `window.scrollBy` does **not** advance the loader — only real wheel/scroll input does.
- **Reliable per-page capture pattern:** click the pager page → wait ~3 s → scroll the results column to the **top** → harvest the 25 now-rendered cards (anchors are present right after a fresh page render; they detach again if you scroll far). "Similar properties" carousels are injected between batches (skip them — require the card to sit inside `.tupleNew__contentWrap`).
- **Filters survive pager navigation** within a session (the result header keeps the full filter string across pages), even though they never reach the URL — so you can walk page 1→20 without re-applying the rail. A **reload** still drops them (see "not deep-linkable").
- **Pace it.** Sweeping all 20 pages fast trips a rate-limit (see Gotchas). Page through deliberately.
- **Open a listing's `-spid-` detail URL** only for the detail-only fields (below) — and open them in **small, spaced batches** (rapid detail hits trip a CAPTCHA; see Gotchas). A full run is dominated by one detail-page open per listing.

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
- **Rate-limit on fast pagination** (seen 2026-06-18): paging through ~15 SRP pages quickly made the results pane return **empty skeleton placeholders** and the pager stopped rendering; a pause did not recover it in-session. Page deliberately, not in a rapid sweep.
- **CAPTCHA on rapid detail-page hits** (seen 2026-06-18): opening several `-spid-` detail pages in quick succession redirected to `…/load/verifycaptcha?redirectionPath=…`. This is an anti-bot wall — **do not solve/bypass** (`docs/rules.md`); it blocks the detail-only fields (incl. `tenant_preference`). Mitigate by spacing detail-page opens; if it fires, stop and resume later. (Supersedes the trial's "no CAPTCHA/rate-limiting" note, which only held for a tiny session.)
- **`tenant_preference` is detail-only & costly at scale** — it never appears on the card, so confirming it for a large result set means one detail open each (and risks the CAPTCHA). When that's infeasible, the **listing description** is the fallback signal for single-gender restrictions ("ladies/girls/women only", "gents/men only", a "…Ladies PG" building name); flag those `[Inference]` and confirm a sample. The applied filter already guarantees bachelor-eligibility.

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| 2026-06-18 | First-contact trial complete: full logged-out browsing; filtered search **not** deep-linkable; scam interstitial per SRP; phone login-walled; 1 RK/1 BHK bundled; only rent + (sometimes) deposit on cost. 3 listings captured + validated; `calc_price_per_sqft` (carpet) added. | Recipe filled. Open: whether maintenance/brokerage ever appear on some listings [Unverified]; whether the unfiltered SRP can be filtered via on-page sort+scan more cheaply than re-applying the rail each session. |
| 2026-06-18 | Full run (`data/2026-06-18-99acres-blr-1bhk-furnished-bachelors`, 493 results, furnished bachelors M+W). **Pagination is a numbered 20-page pager (~25/page), not pure infinite scroll**; list is **windowed/virtualised** (≈25 cards in DOM; `-spid-` anchor attaches only after the card's photo lazy-loads). Working capture: click pager page → wait → scroll results to top → parse 25. **Filters survive pager nav** (but not reload). **Rate-limit** (empty skeletons) hit ~page 16 after fast paging; **CAPTCHA** (`/load/verifycaptcha`) fired on rapid detail-page hits → not bypassed, capped tenant_preference sample to 3 (all "All"). Captured 350/493 (pages 1–15). | Update recipe: pace pagination (don't sweep all 20 fast); capture detail fields in small spaced batches to dodge the CAPTCHA; treat windowed virtualisation + lazy anchors as the model; numbered pager is the reliable traversal. Tenant rule applied via filter + description scan (4 single-gender flagged [Inference]). |
