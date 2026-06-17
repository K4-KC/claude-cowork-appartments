# CSV data schema

Each run writes one CSV per site at `data/<run-id>/<site>.csv`. The columns below are a **starting proposal** — refine as real captures reveal what each site does and doesn't expose. Keep column names identical across sites so the per-site tables stay comparable.

## Columns (proposed)

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

Computed columns (from `docs/calculations.md`) are appended once calculations are defined.

## Conventions

- **Currency:** plain integers in rupees, no separators or symbols (e.g. `32000`, not `₹32,000`).
- **Missing values:** leave the cell empty rather than guessing. If a value is uncertain or approximate, note it (e.g. in a `notes` column) rather than presenting it as exact.
- **Don't fabricate:** only record what Cowork actually captured from the page.
- **Raw vs processed:** if you keep an untouched capture alongside the processed table, agree on a naming convention (e.g. `<site>.raw.csv`) and document it here before using it.
