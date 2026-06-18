# Findings — 99acres run `2026-06-18-99acres-blr-1bhk-furnished-bachelors`

Normal run (recipe already existed). Filters: Bengaluru · Rent · 1 RK/1 BHK · ₹12,000–22,000 · Furnished · Available-for **Single Women + Single Men**. Goal: all matching listings, 1 RK studios included+flagged, tenant rule = both men & women OR "All" (exclude single-gender).

- **Filled by:** Cowork (Claude in Chrome) · **Date:** 2026-06-18 · **Status:** partial (see coverage)

## Outcome

- Live filtered result count: **493**. **Captured: 350** unique listings (pages 1–15 of 20). Written to `99acres.csv` (350 rows, full schema + `calc_` columns). Summary in `comparison.md`.
- **Not captured: pages 16–20 (~143 listings)** — see "What diverged".

## What diverged from `docs/sites/99acres.md` (feed back into the recipe)

1. **Pagination is NOT pure infinite scroll.** The SRP now exposes a **numbered pager (20 pages, ~25/page)** with a "Next Page >" link (`<a class="list_header_bold">`) and page-number anchors (`<a class="Pagination__active">` marks current). The list is **windowed/virtualised**: only ~25 cards exist in the DOM at once, and a card's `-spid-` anchor only attaches once its photo lazy-loads. Reliable capture pattern that worked: click a pager page → wait ~3 s → scroll the results column to the top → parse the 25 rendered cards (their anchors are present right after a fresh page render). `window.scrollBy` did **not** advance the loader; only real wheel/scroll events did.
2. **Filters DO survive pager navigation** even though they never reach the URL (header kept "493 results | 1 BHK Fully Furnished … for Single Women, Single Men Below 22000" across pages). So within one session you can page 1→20 without re-applying filters — but a reload still drops them (confirms recipe's "not deep-linkable").
3. **Rate-limit after rapid pagination.** Around page 16 the results pane began returning **empty skeleton placeholders** and the pager stopped rendering — a throttle from paginating too fast. Pausing did not recover it within the session.
4. **CAPTCHA on rapid detail-page hits.** Opening several `-spid-` detail pages in quick succession redirected to `…/load/verifycaptcha?redirectionPath=…`. **Not solved/bypassed** (per `docs/rules.md`). This capped tenant_preference confirmation to a 3-listing sample (all "Available For: All").
5. Scam interstitial + cookie notice fired on first SRP load (dismissed "Ok, understood" / "Okay"), consistent with the recipe.

### Recommended recipe update
Pace pagination like a human (a few seconds between pages; don't sweep all 20 fast), and capture detail-only fields in small, spaced batches to avoid the CAPTCHA. Treat 25-card windowed virtualisation + lazy anchors as the model. Numbered pager is the reliable traversal (not scroll-append).

## Tenant rule — how it was applied (important)

99acres shows tenant preference **only on the detail page** ("Available For: All / Bachelors (Men/Women) / Family"), not on cards. With 493 results, per-listing confirmation = 493 detail opens (infeasible; and the CAPTCHA blocked even a large sample). Applied instead:
- The **filter** already restricts to bachelor-eligible (Single Women ∪ Single Men).
- **Detail sample (3):** P90503832, Y91659522, G88037698 → all **"All"**.
- **Description scan (all 350)** for single-gender wording → **4 flagged** (kept in CSV, flagged, **[Inference] not verified**; the 3 clear ones excluded from the both/all set):
  - `A91757048` (CV Raman Nagar) — "men only"
  - `K90111822` ("SL Ladies PG", Bellandur) — ladies PG / women-only
  - `W51505944` (Jayanagar) — "Only women"
  - `H89355255` (Neeladri Nagar) — both phrasings, review

So **347 of 350 match** the "both men & women / all" rule on available signals; tenant_preference column is left blank except the 3 confirmed (don't treat blanks as "All").

## Field coverage this run

Cards reliably gave: `listing_id` (spid), `url`, `title`, `building_name`, `locality` (parsed from title), `rent`, `deposit` (₹ or "N months rent"), `area` + `area_basis` (carpet/built-up/super/**plot**), `bathrooms`, `posted_by`, `property_type` (from title), and the 1 RK-vs-1 BHK distinction. Detail-only fields (`address`, `floor`, `property_age`, `parking`, `amenities`, `available_from`, `tenant_preference`, `contact_name`) are **blank** — detail sweep was cut by the CAPTCHA. `contact_phone` login-walled. `maintenance/maintenance_included/brokerage/move_in_charges` not exposed by 99acres (blank, absent).

## Data-quality flags (captured as-is, not corrected)

- **110 / 350 are 1 RK studios** (bundled filter) — flagged in `notes`, not true 1 BHK.
- **54 / 350 list Plot Area** (land, not living area) — some absurd (e.g. 13,500 / 6,000 sqft on a 1 BHK); ₹/sqft on these is meaningless, excluded from the value ranking.
- A few **implausible deposits** (₹1, ₹40, ₹2,600) — kept, flagged.
- Deposits given as "N months rent" were normalised to ₹ (N × rent) and noted.

## To finish the set later
Resume pages 16–20 (~143 listings) in a fresh, slower-paced session, and detail-enrich tenant_preference in small spaced batches (avoid the CAPTCHA). Merge into this CSV by `listing_id`.
