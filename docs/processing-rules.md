# Cross-site processing rules

Validation and normalization rules the **trial round** surfaced (Housing.com, 99acres, NoBroker; Magicbricks was blocked — see its recipe). They apply in the **processing step** here in Claude Code (`docs/workflow.md` steps 4–5): how to validate captured CSVs and how to *read* captured columns when computing derived `calc_` columns. They never rewrite captured values (`docs/rules.md`).

Sites stay separate (`docs/calculations.md`) — these rules make *within-site* comparison sound and flag where *cross-site* comparison is and isn't valid.

## 1. Area basis — normalize before comparing ₹/sq.ft
`area` is captured on whatever basis the site shows, recorded in `area_basis`:
- Housing.com → **built-up**
- NoBroker → **built-up**
- 99acres → **carpet** (and sometimes **"Plot Area"** = land, not living area — not comparable; capture the living-area basis and flag)

`calc_price_per_sqft` is only comparable across rows with the **same `area_basis`**. Built-up and carpet ₹/sq.ft differ materially (carpet is smaller → higher ₹/sq.ft for the same rent). When a table mixes bases, segment by basis or flag it. Do **not** silently convert one basis to another.

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
99acres sometimes gives deposit as "N months rent" rather than a number; NoBroker may note "negotiable". Where it's a multiple, convert to ₹ using `rent` and note it; where only "negotiable"/absent, leave as captured/blank — don't invent a figure.

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

## Identity columns (reference, for any future dedup)
`listing_id` source differs by site: Housing = Property ID in the detail URL; 99acres = the `spid`; NoBroker = the hex id in `…/{id}/detail`. All stable and deep-linkable.
