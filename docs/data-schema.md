# CSV data schema

Each run writes one CSV per site at `data/<run-id>/<site>.csv`. The captured columns below are the **agreed set** for a run, chosen from what the search is meant to surface (cost, space, location, and condition). Treat them as the target schema: a site that doesn't expose a given field leaves that cell blank (see conventions) rather than dropping the column. Keep column names identical across sites so the per-site tables stay comparable.

## Two kinds of columns: captured vs derived

Every column in a CSV is one of two kinds, and they must stay clearly separable:

- **Captured columns** — taken **directly from the site** by Cowork while browsing. These are the source of truth. Processing here may validate and flag them, but **never rewrites** their values (see `docs/rules.md`).
- **Derived columns** — **computed here** in Claude Code during post-processing (the calculations in `docs/calculations.md`). They are added on top of captured data; they are not present in the original capture.

To keep the two visually distinct in the CSV, **derived columns are prefixed `calc_`** (e.g. `calc_true_monthly_cost`, `calc_price_per_sqft`). Captured columns use plain names. Derived columns are appended **after** all captured columns and never overwrite one.

## Captured columns (written directly by Cowork)

Grouped for readability; in the CSV they appear in this order. Every field is taken from the listing as shown, except the bookkeeping fields (`listing_id`, `source_site`, `url`, `captured_at`), which Cowork stamps at capture time.

### Identity & source

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `listing_id` | string | — | Site's own id if available |
| `source_site` | string | — | housing / 99acres / nobroker / magicbricks |
| `url` | string | — | Direct link to the listing |
| `title` | string | — | Listing headline as shown |
| `captured_at` | datetime | ISO 8601 | When Cowork captured the row |

### Location

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `building_name` | string | — | Society / project / apartment / building name. Basis for Google Maps geocoding in derived data later — capture even when only partial. |
| `locality` | string | — | Neighborhood / area |
| `city` | string | — | |
| `address` | string | — | Street address or nearest landmark, as listed (often approximate); aids the map lookup |

### Property & layout

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `property_type` | string | — | Captured raw; normalized on read to the enum apartment / independent / villa / PG / studio (see `docs/processing-rules.md` §6) |
| `bhk` | number | — | 1, 2, 3, … |
| `bathrooms` | number | — | Count, if listed |
| `area` | number | sq.ft | Note basis in `area_basis` |
| `area_basis` | string | — | carpet / built-up / super |
| `floor` | string | — | e.g. "3 of 12" — encodes both the floor and the total floors |
| `furnishing` | string | — | Single enum: unfurnished / semi / full. **Settled** — one column, not split (it is one categorical value, not a list). See below. |
| `property_age` | string | — | As listed, e.g. "5 years" or "built 2018" |
| `parking` | string | — | As listed, e.g. "1 covered car; bike" |
| `amenities` | string | — | One semicolon-separated list of raw tokens as the site shows them. **Settled** — single column at capture; amenity-level access comes from derived `calc_amenity_*` flags. See below. |

> **Settled — amenities & furnishing representation (2026-06-18).** Both stay **single columns at capture**:
> - **`furnishing`** is one categorical value (unfurnished / semi / full), so there is nothing to split — keep it as one enum column. (If a run ever needs the detailed furnished-items inventory — AC, fridge, bed, … — that is a *new* capture field and would follow the same single-column-then-derive rule as amenities; it is not captured today.)
> - **`amenities`** stays one semicolon-separated list of the **raw tokens exactly as the site shows them** (source of truth, never rewritten). We do **not** break amenities into one column per amenity at capture time. The trial round showed why: the four sites share almost no amenity vocabulary ("Gated Society" vs "Gated Community" vs "Gated Security: No"), encode differently (NoBroker uses `key: value` pairs), and smuggle non-amenities into the list (facing direction, metro-proximity — both schema-excluded). A per-amenity capture schema would be a sparse, mostly-blank wide table that is *not* comparable across sites, and would force Cowork to normalize while browsing (which violates the captured-vs-derived split).
> - **Amenity-level filtering/scoring** is served instead by derived **`calc_amenity_*` boolean flags** computed here in Claude Code from a synonym/normalization map — only for the handful of amenities a run actually filters on, not all of them. See the derived-columns section below, `docs/calculations.md`, and the amenity-normalization map in `docs/processing-rules.md`.

### Cost

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `rent` | number | ₹/month | Base monthly rent |
| `deposit` | number | ₹ | Security deposit |
| `maintenance` | number | ₹/month | Separate maintenance if listed |
| `maintenance_included` | boolean | true / false | Whether maintenance is already part of rent (blank if unclear) |
| `brokerage` | number | ₹ | Broker commission / one-time fee (typically 0 for owner listings) |
| `move_in_charges` | number | ₹ | Other one-time / move-in / society charges, if listed |

### Availability & tenancy

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `available_from` | date | YYYY-MM-DD | Move-in availability. Captured as shown: an ISO date when the site gives one, otherwise a raw availability token ("Immediate" / "Available now" / "Immediately"). Normalized in processing — see `docs/processing-rules.md` §3. |
| `tenant_preference` | string | — | family / bachelors / company / any |
| `posted_by` | string | — | owner / broker |

### Contact

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `contact_name` | string | — | Owner's or agent's name, as listed |
| `contact_phone` | string | — | Phone/contact as listed, only where the site shows it without requiring a login (see `docs/rules.md`). String, to preserve formatting. |

### Notes

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `notes` | string | — | **Empty by default.** Free-text, filled only when a row needs a caveat or an uncertain/approximate value flagged (see conventions). |

### Considered but excluded

Decided against for now (add later if a run needs them): **facing direction**, **photo/image links**, **posted/updated date**, and a **rent-negotiable** flag.

## Derived columns (computed here during post-processing)

These do **not** come from the sites — Claude Code adds them after capture by applying `docs/calculations.md` to the captured columns above. Each is defined once its calculation is. The columns defined so far — `calc_price_per_sqft`, `calc_true_monthly_cost` (with its `calc_cost_basis` flag), and the on-demand `calc_amenity_*` flags — are listed below; more are added as calculations are decided in `docs/calculations.md`.

| Column | Type | Unit / format | Derived from |
|---|---|---|---|
| `calc_price_per_sqft` | number | ₹/sq.ft/month | `rent ÷ area`, on the captured `area_basis` — see `docs/calculations.md` |
| `calc_true_monthly_cost` | number | ₹/month | `rent + extra maintenance + (brokerage+move-in)/12 + deposit×0.005` — see `docs/calculations.md` |
| `calc_cost_basis` | string | full / lower-bound | Confidence flag for `calc_true_monthly_cost`: `full` when every additive cost (`maintenance`, `brokerage`, `move_in_charges`) is captured or confirmed ₹0; `lower-bound` when ≥1 is unknown (site/listing never exposed it) so the value understates true cost — judged on real-world completeness, not on what a site exposes. See `docs/calculations.md`. |
| `calc_amenity_<name>` | boolean | true / false | **On-demand.** One flag per amenity a run filters on (e.g. `calc_amenity_gated`, `calc_amenity_lift`, `calc_amenity_security`), derived from the raw `amenities` list via the synonym map in `docs/processing-rules.md`. Only the amenities in use are materialized — not one column per possible amenity. `false` = absent/not mentioned (not "site doesn't list amenities"); leave blank only when `amenities` itself was uncaptured. |
| `calc_*` | — | — | _Added per calculation once defined in `docs/calculations.md`._ |

Rules for derived columns:

- Prefix every derived column `calc_` so it is self-evidently post-processed, not captured.
- Append them after the captured columns; never overwrite or edit a captured value to produce one.
- If a derived value can't be computed (a required captured input is missing), leave it empty — don't guess.

## Conventions

- **Currency:** plain integers in rupees, no separators or symbols (e.g. `32000`, not `₹32,000`).
- **Missing values:** leave the cell empty rather than guessing. If a value is uncertain or approximate, note it (e.g. in a `notes` column) rather than presenting it as exact.
- **Don't fabricate:** only record what Cowork actually captured from the page. Computed values belong in `calc_` columns, never written back into a captured column.
- **Raw vs processed file:** the per-site CSV holds captured columns plus appended `calc_` columns, so capture and derivation are distinguishable within one file. If you additionally want an untouched capture-only file alongside the processed one, agree on a naming convention (e.g. `<site>.raw.csv`) and document it here before using it.
