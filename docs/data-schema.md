# CSV data schema

Each run writes one CSV per site at `data/<run-id>/<site>.csv`. The columns below are a **starting proposal** — refine as real captures reveal what each site does and doesn't expose. Keep column names identical across sites so the per-site tables stay comparable.

## Two kinds of columns: captured vs derived

Every column in a CSV is one of two kinds, and they must stay clearly separable:

- **Captured columns** — taken **directly from the site** by Cowork while browsing. These are the source of truth. Processing here may validate and flag them, but **never rewrites** their values (see `docs/rules.md`).
- **Derived columns** — **computed here** in Claude Code during post-processing (the calculations in `docs/calculations.md`). They are added on top of captured data; they are not present in the original capture.

To keep the two visually distinct in the CSV, **derived columns are prefixed `calc_`** (e.g. `calc_true_monthly_cost`, `calc_price_per_sqft`). Captured columns use plain names. Derived columns are appended **after** all captured columns and never overwrite one.

## Captured columns (written directly by Cowork)

| Column | Type | Unit / format | Notes |
|---|---|---|---|
| `listing_id` | string | — | Site's own id if available |
| `url` | string | — | Direct link to the listing |
| `title` | string | — | As shown on the site |
| `locality` | string | — | Neighborhood / area |
| `city` | string | — | |
| `bhk` | number | — | 1, 2, 3, … |
| `property_type` | string | — | apartment / independent / villa / PG |
| `rent` | number | ₹/month | Base monthly rent |
| `deposit` | number | ₹ | Security deposit |
| `maintenance` | number | ₹/month | Separate maintenance if listed |
| `area` | number | sq.ft | Note carpet vs built-up in `area_basis` |
| `area_basis` | string | — | carpet / built-up / super |
| `furnishing` | string | — | unfurnished / semi / full |
| `floor` | string | — | e.g. "3 of 12" |
| `available_from` | date | YYYY-MM-DD | |
| `posted_by` | string | — | owner / broker |
| `amenities` | string | — | Semicolon-separated list |
| `captured_at` | datetime | ISO 8601 | When Cowork captured the row |
| `source_site` | string | — | housing / 99acres / nobroker / magicbricks |

> `captured_at` and `source_site` are recorded by Cowork at capture time, so they count as captured columns even though Cowork generates them rather than reading them off the listing.

## Derived columns (computed here during post-processing)

These do **not** come from the sites — Claude Code adds them after capture by applying `docs/calculations.md` to the captured columns above. None are committed yet; each is defined once its calculation is. Until then this section stays empty.

| Column | Type | Unit / format | Derived from |
|---|---|---|---|
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
