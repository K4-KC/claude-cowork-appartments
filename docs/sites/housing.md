# Recipe: Housing.com (housing.com)

> **Cowork — start here.** This recipe is self-contained: to run Housing.com you need only this file. First read **`docs/cowork-run.md`** (the standing run procedure), then follow the site-specific steps below. **Read order:** `docs/cowork-run.md` → this file → `docs/data-schema.md` → `docs/rules.md`. A normal run needs **no kickoff message** — kickoffs are only for first-contact trials of a not-yet-reciped site (`docs/trial-protocol.md`).

The durable run recipe for Housing.com, distilled from the first-contact trial. If the live site has drifted from what's below, capture the difference in the run's `findings.md` so this recipe can be refreshed.

## Trial status

- **Status:** First-contact trial **complete** (run `data/2026-06-18-trial-housing-blr-1bhk/`, 2026-06-18). Raw findings: that run's `findings.md`.
- Sections below are distilled from that trial. Codes/URLs marked **[Inference]** were seen in a single session — reproduce, don't trust blindly.

## URL / entry point

- **Base search URL** (Bengaluru rent flats, no filters): `https://housing.com/rent/flats-for-rent-in-bangalore-karnataka-P38f9yfbk7p3m2h1f` — `P38f9yfbk7p3m2h1f` is the base search id for "Bengaluru rent flats".
- **Deep-linkable? Yes** (reloading a fully-filtered URL in a fresh tab reproduced the exact result set — verified). **But the encoding is an opaque token scheme in the URL path**, not human-readable `?key=value` query params:
  - A single filter yields a readable slug, e.g. 1 BHK only → `https://housing.com/rent/1bhk-flats-for-rent-in-bangalore-karnataka-C2P38f9yfbk7p3m2h1f`.
  - Combining filters switches to a `search-<codes>` form, e.g. the trial's four filter codes (BHK, furnishing, lease, budget) on the base id → `https://housing.com/rent/search-C2G1L4P38f9yfbk7p3m2h1fT99cUgz4`.
  - Observed codes **[Inference]**: `C2` = 1 BHK, `G1` = Fully Furnished, `L4` = lease type Bachelor, `T99cUgz4` = budget ₹12k–22k (opaque; decode unverified).
- **Recommended approach:** don't hand-build the token. Apply filters once in the UI, copy the resulting URL, reuse *that* for the run. Treat it as opaque-but-stable-within-a-run.

## Applying the run's filters

| Filter | How to set on Housing.com | Notes |
|---|---|---|
| City | Homepage city selector / search box; baked into the base id (`P38f9yfbk7p3m2h1f` = Bangalore). Path also carries readable `...-in-bangalore-karnataka-...`. | Homepage defaulted to Bengaluru. |
| Listing type (Rent) | "RENT" tab on homepage → `/rent/` path | Clean. |
| BHK | Top filter bar → "BHK Type" → 1 BHK (`C2`) | Options 1 RK … 5+ BHK. **Quirk: applying budget cleared the BHK filter** — re-check after each filter (or set budget first, BHK last). |
| Budget | Top filter bar → budget → Min/Max numeric inputs (slider too); token `T99cUgz4` | Free numeric entry → any range expressible. |
| Furnishing | Top filter bar → "Furnishing" → Fully Furnished (`G1`) | Options Fully / Semi / Unfurnished. |
| Tenant / bachelors | Top filter bar → "More Filters" → "Lease Type" → Bachelor (`L4`) | Single **Bachelor** option (not split male/female), plus Family, Company. Maps cleanly to "bachelors"/"any". |

All six trial filters were expressible in the UI — none deferred to processing.

## Walking results

- **Infinite scroll.** First batch ≈ 30 cards ("Showing 1–30 of N properties"); scrolling auto-appends more. **The header count is static** and does not reflect how many cards have actually loaded. No numbered pages, no "load more" button.
- **Recommendation carousels** ("Newly added…", featured agents) are **injected between batches** — the list is not one continuous block; skip these when harvesting.
- **Cost set lives in a popover:** each card has a "see price breakup" popover with rent / maintenance / deposit / brokerage / painting (move-in) — **open it for every card** to capture the cost columns uniformly.
- **Open a listing's detail page** only for the detail-only fields (below). A full run is dominated by one detail-page open per listing.

## Extraction → CSV

Cowork captures only the **captured columns** in `docs/data-schema.md` directly from the page. The derived `calc_` columns are added later by Claude Code in processing — never produce them here.

**Field-coverage map** (`card` = results card · `card(breakup)` = price-breakup popover · `detail` = detail page only · `login` = login/lead-gated gap):

- **On the card:** `title`, `locality`, `city`, `rent`, `bhk`, `property_type`, `area` + `area_basis` (labelled "Builtup area"), `furnishing`, `posted_by`, `contact_name`, `url`.
- **In the card price-breakup popover:** `deposit`, `maintenance`, `maintenance_included` ([Inference] — separate line ⇒ not included), `brokerage`, `move_in_charges` ("Painting Charges").
- **Detail page only:** `bathrooms`, `floor` ("3 of 5 floors" = floor + total), `property_age` ("0 year"/"5 years"), `parking` (exact), `available_from` ("Available now"), `tenant_preference` (lease type, slash-set), full `address`, `building_name` (society/project, when present), full `amenities` list, `listing_id` (Property ID embedded in the detail URL). Card amenities are a curated subset only.
- **Login-walled (gap, leave blank):** `contact_phone` — masked behind a "Get Contact Details" lead form on every detail page.
- **Variable:** `building_name` is absent for independent listings with no named society.

## Gotchas

- **`contact_phone` is login/lead-gated** on every listing (masked `+9198869.....` behind a name+phone form). Recorded as a gap, left blank — not worked around.
- **Budget filter silently clears the BHK filter** mid-setup. Always re-check the breadcrumb + result count after each filter, or apply budget first and BHK last.
- **Infinite scroll + injected carousels** make bulk card-loading tedious, and the "of N" header is static — don't trust it for progress.
- **Cost columns require opening the price-breakup popover** per card; otherwise deposit/maintenance/brokerage/move-in are blank.
- **`available_from` is a text token** ("Available now"), not a date — processing normalizes it.
- No CAPTCHA, rate-limiting, or forced popups seen during the trial (logged out).

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| 2026-06-18 | First-contact trial complete: deep-linkable (opaque token URLs), full logged-out browsing, `contact_phone` the only wall, infinite scroll, cost set in a popover, ~9 detail-only fields. 3 listings captured + validated; `calc_price_per_sqft` added. | Recipe filled. Open: owner-listing pattern unsampled (all 3 were brokers); confirm token-URL stability across sessions in a real run. |
