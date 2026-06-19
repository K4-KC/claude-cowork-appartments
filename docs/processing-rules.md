# Cross-site processing rules

Validation and normalization rules the **trial round** surfaced (Housing.com, 99acres, NoBroker; Magicbricks was blocked — see its recipe). They apply in the **processing step** here in Claude Code (`docs/workflow.md` steps 4–5): how to validate captured CSVs and how to *read* captured columns when computing derived `calc_` columns. They never rewrite captured values (`docs/rules.md`).

Sites stay separate (`docs/calculations.md`) — these rules make *within-site* comparison sound and flag where *cross-site* comparison is and isn't valid.

## 1. Area basis — normalize before comparing ₹/sq.ft
`area` is captured on whatever basis the site shows, recorded in `area_basis`:
- Housing.com → **built-up**
- NoBroker → **built-up**
- 99acres → **basis varies per listing** — not a single basis. A 350-listing run (2026-06-18) was carpet 41% / super built-up 24% / built-up 20% / **plot 15%**. "Plot Area" = land, not living area (not comparable; flag it). So **segment by `area_basis` even within a single 99acres run**, not just across sites. (Earlier note said "99acres → carpet"; that was a 3-listing sample — basis is mixed.)

`calc_price_per_sqft` is only comparable across rows with the **same `area_basis`**. Built-up and carpet ₹/sq.ft differ materially (carpet is smaller → higher ₹/sq.ft for the same rent). When a table mixes bases, segment by basis or flag it. Do **not** silently convert one basis to another.

**Guard ₹/sq.ft rankings against poster-error areas.** Owner/dealer-entered `area` is often wrong — a 1 BHK / 1 RK listing with `area` of 1,500 / 6,000 / 13,500 sqft is a data-entry error that silently produces an absurdly *low* ₹/sq.ft and poisons any "best value" sort. When ranking by `calc_price_per_sqft`, restrict to a **plausible living-area band** (e.g. ~250–900 sqft for 1 RK/1 BHK) and exclude `area_basis = plot`; keep all rows in the CSV but don't let outliers top the ranking. Flag the implausible ones in `notes` rather than rewriting them.

## 2. Cost-field availability — per-site map (drives `calc_true_monthly_cost`)
Absence of a cost field ≠ ₹0. What each site actually exposes:

| Cost column | Housing.com | 99acres | NoBroker |
|---|---|---|---|
| `rent` | ✅ | ✅ | ✅ |
| `deposit` | ✅ | sometimes | ✅ |
| `maintenance` / `maintenance_included` | ✅ (price-breakup popover) | ✗ not shown | ✅ (card) |
| `brokerage` | ✅ (popover) | ✗ | ✅ = **0** (owner platform) |
| `move_in_charges` | ✅ (popover; "Painting") | ✗ | ✗ |

So set `calc_cost_basis = full` only when every **additive** cost (`maintenance`, `brokerage`, `move_in_charges`) was captured **or confirmed ₹0**; otherwise `lower-bound` (judged on real-world completeness, **not** on what a site happens to expose — see `docs/calculations.md`). Concretely, across the trial sites:
- **99acres** → `lower-bound`: it never shows maintenance/brokerage/move-in, so the figure (`rent` + deposit opportunity) is a floor.
- **NoBroker** → `lower-bound`: brokerage is a confirmed 0 and maintenance is shown, but **`move_in_charges` is never exposed**, so one additive cost stays unknown.
- **Housing** → **open the price-breakup popover** to capture maintenance/brokerage/move-in; a row is `full` only when all three are captured, else `lower-bound`.

## 3. `available_from` — token vs date, and poster errors
Captured as shown; values are mixed:
- Tokens: "Available now" (Housing), "Immediate"/"Ready to move" (99acres), "Immediately"/"Ready to Move" (NoBroker) → normalize to an `immediate` sentinel (or the capture date) **on read** when filtering/computing — don't rewrite the captured cell.
- Dates: NoBroker "Possession: Jul 1, 2026" → `2026-07-01`.
- **Poster errors happen:** a 99acres listing showed `2030-11-30` while also saying "Ready to move". Treat far-future / self-contradictory dates skeptically; don't feed them blindly into availability or commute logic.

## 4. `tenant_preference` is an allowed-SET, not one value
Sites list which tenant types are permitted, variously joined:
- Housing: "Family/Company/Bachelor" (slash set)
- 99acres: "All" / "Bachelors (Men/Women)"
- NoBroker: often "Anyone" even under a Bachelor-Male+Female filter (bachelor-eligible ⊇ Anyone)

Filter/score on **membership** ("is bachelor allowed?"), never equality. A bachelor search does not imply the listing prints "Bachelor".

**99acres: `tenant_preference` is detail-page-only and may be unconfirmable at scale.** It never shows on the card, so confirming it for a large set means one detail open per listing — and rapid detail opens trip a CAPTCHA (`docs/sites/99acres.md`). When a run needs a *gender* split (e.g. "both men & women / all, exclude single-gender") and full detail capture isn't feasible:
- Lean on the **filter** first — selecting Single Women + Single Men already restricts to bachelor-eligible (men ∪ women).
- **Screen the listing description** for single-gender wording — "ladies/girls/women/female only", "gents/men/male/boys only", or a tell in the building name ("…Ladies PG"). Flag matches `[Inference]` (a heuristic, **not** the site's tenant field) and **confirm a sample** of detail pages where possible.
- Leave the `tenant_preference` column **blank** for un-confirmed rows (don't backfill "All" from the filter); record the gender flag in `notes`. Treat blanks as unknown, not as "All".

## 5. `posted_by` reliability varies
- **Normalize on read** (don't rewrite the captured cell; case-insensitive) to the schema enum `owner / broker`: 99acres `Dealer` → broker, `Owner` → owner; Housing agency name → broker, `Owner` → owner.
- Housing.com / 99acres: **printed** (owner/broker, owner/dealer) — trust it.
- NoBroker: **inferred** — no per-listing owner/broker label logged out; "owner" is the platform default (`brokerage = 0`). Treat NoBroker `posted_by=owner` as `[Inference]`, not observed.

## 6. `property_type` — normalize variants to the enum
Map captured wording to `apartment / independent / villa / PG / studio` on read (don't rewrite the captured cell):
- "Flat", "Apartment" → apartment
- "Independent Builder Floor", "Builder Floor", "independent floor" → independent
- "Studio Apartment (1 RK)" → studio (⚠ also a BHK-ambiguity flag — see §7)

## 7. BHK ambiguity (99acres)
99acres bundles **1 RK and 1 BHK** under one filter button, so studios appear in "1 BHK" results with `bhk=1`. If a run needs *true* 1 BHK, exclude studios in processing (flagged in `notes`/title). Housing and NoBroker keep 1 RK separate.

## 8. `deposit` expressed as a rule
99acres sometimes gives deposit as "N months rent" rather than a number; NoBroker may note "negotiable". Where it's a multiple, convert to ₹ using `rent` and note it; where only "negotiable"/absent, leave as captured/blank — don't invent a figure. **Implausible captured deposits happen** (a 99acres run saw `₹1`, `₹40`, `₹2,600` against five-figure rents — clearly poster placeholders); keep them as captured and **flag in `notes`**, don't silently "fix". They drag `calc_true_monthly_cost` (the `deposit×0.005` term) toward the rent floor, which is consistent with its `lower-bound` basis on 99acres anyway.

## 9. Run-scope quirks (affect how a run is defined, not a single row)
- **NoBroker has no city-wide search** — it requires a locality geo-token; browse per-locality (also in `docs/search-config.md`). Mind its default `radius=2.0 km`, which pulls in neighbouring localities (a "Koramangala" search surfaced a BTM Layout listing) — filter by locality if strictness matters.
- **Deep-linkability differs:** Housing = opaque token URL (reusable); 99acres = filters never reach the URL (re-apply each session); NoBroker = token URL but the locality token is mandatory. Mint token URLs via the UI once and reuse; don't hand-build tokens.

## 10. Amenity normalization (drives `calc_amenity_*` flags)
`amenities` is captured as one semicolon-separated list of raw site tokens (settled — see `docs/data-schema.md`). The four sites barely share a vocabulary, so amenity-level flags are **derived here** by mapping each site's tokens to a canonical flag. Match case-insensitively and substring-aware; **handle NoBroker's `key: value` form and negations** (`Gated Security: No` → gated = **false**).

Seed map from the trial round (extend as new tokens appear; only materialize flags a run actually filters on):

| Canonical flag | Synonyms / tokens seen (by site) | Notes |
|---|---|---|
| `calc_amenity_gated` | "Gated Society" (99acres), "Gated Community" (housing), "Gated Security: Yes/No" (nobroker) | NoBroker form is `key: value` — read the value; `No` → false |
| `calc_amenity_lift` | "Lift" (housing) | |
| `calc_amenity_power_backup` | "Power Backup" (housing) | |
| `calc_amenity_security` | "CCTV" (housing), "Gated Security" (nobroker) | CCTV ≈ surveillance; decide per run whether CCTV alone counts |
| `calc_amenity_parking` | "Visitor Parking" / "Parking Available" (99acres) | Cross-check the captured `parking` column too |
| `calc_amenity_piped_gas` | "Piped-gas" (99acres) | |
| `calc_amenity_water_supply` | "Regular Water Supply" (housing), "Water Supply: Borewell/Both/Corporation" (nobroker) | NoBroker reports the *source*; presence ≠ "regular" |

**Not amenities — do not map** (they ride along in some sites' lists but belong to excluded/other fields): facing direction ("North-East Facing"), location proximity ("Close to Metro/Market/Hospital/School"), property attributes ("Corner Property", "Vaastu Compliant"), flooring ("Vitrified Flooring"). Ignore these when deriving amenity flags.

## 11. Google Maps geo reads (drives `geo.csv`)
Geo data is **captured from Google Maps** into `data/<run-id>/geo.csv` (schema in `docs/data-schema.md`; capture method in `docs/sites/google-maps.md`). Rules for reading/normalizing it here:

- **Unit/time normalization.** Distance is `km` ≥1 km but **metres** below (`550 m` → `0.55`). Time: `33 min`, `1 hr 8 min` → `68`, `2 hr 24 min` → `144` → store integer minutes.
- **`*_mm_*` (transit).** `*_mm_min` = **transit time as Maps shows it — bus *or* metro, whichever is fastest, NOT pure walk+metro** (short inner hops are usually bus; metro wins only long-haul). `*_mm_km` is **permanently blank** (Maps' transit panel shows no trip distance) — don't treat blank as a capture miss.
- **Route-selection rule.** Maps' **"Best route"** is occasionally not the shortest distance. Trial used **top/first-listed** for car, **shortest duration** for transit — *standardize before the full run* (open decision in the recipe); whichever is chosen, apply it uniformly so `calc_price`-style rankings stay comparable.
- **Trust the resolved place, not the query.** A POI query can silently resolve to the **wrong city** (a Nelamangala MedPlus resolved near Mysore, ~150 km). Verify the resolved title/coords; flag any distance whose destination looks out-of-region.
- **`geo_confidence` tiers.** `building` = precise; **`locality` = approximate** (centroid — don't rank walk-to-pharmacy precision on these); `none` = lat/lng and all distances blank. Segment or flag locality-tier rows when ranking on fine-grained proximity.
- **Distance is traffic-invariant; times are not.** Trial times were 08:00-IST rush-inflated. Prefer **distance** for ranking; treat captured times as approximate (or standardize the capture hour).
- **Dedup + cache (also a speed rule).** Geocode/capture **once per unique building** (deduped across sites) and reuse across runs via a master geo store; join back to listings on a derived **`calc_geo_key`** (normalized building+locality). A building shared by many listings/sites is one geo row, not many.

## Identity columns (reference, for any future dedup)
`listing_id` source differs by site: Housing = Property ID in the detail URL; 99acres = the `spid`; NoBroker = the hex id in `…/{id}/detail`. All stable and deep-linkable.
