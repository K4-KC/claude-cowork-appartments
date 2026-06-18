# Additional logic / calculations

These produce **derived data** — values computed here in Claude Code from the columns Cowork captured, not anything read off the sites. Each calculation's output is a derived column and follows the derived-column rules in `docs/data-schema.md`: name it with a `calc_` prefix, append it after the captured columns, and never overwrite a captured value to hold the result.

**Status: partially decided.** The user chooses which calculations to run; do not assume any *candidate* below without confirming. **Two calculations are now defined** — `calc_price_per_sqft` and `calc_true_monthly_cost` (with its `calc_cost_basis` flag) — from the trial round. Commute/distance and weighted-ranking remain candidates.

## Defined calculations

### `calc_price_per_sqft`
- **Formula:** `rent ÷ area` (both captured columns), rounded to 2 decimals.
- **Unit:** ₹ per sq.ft per **month** (rent is monthly).
- **Basis:** uses the captured `area` on its captured `area_basis`. Compare like-for-like (e.g. all built-up); built-up and carpet ₹/sq.ft are **not** directly comparable, so flag when a table mixes bases.
- **Missing inputs:** if `rent` or `area` is blank/zero, leave `calc_price_per_sqft` empty — don't guess.
- **First applied:** Housing.com trial `data/2026-06-18-trial-housing-blr-1bhk/`.

### `calc_true_monthly_cost`
Effective ₹/month, spreading one-time and refundable amounts over an assumed tenancy.

- **Formula:** `rent + maintenance_if_extra + (brokerage + move_in_charges) / STAY_MONTHS + deposit × DEPOSIT_OPP_RATE_MONTHLY`, rounded to 2 decimals, where:
  - `maintenance_if_extra` = `maintenance` when `maintenance_included` is `false`; `0` when maintenance is already included in rent.
  - one-time costs (`brokerage`, `move_in_charges`) are amortized over **`STAY_MONTHS = 12`**.
  - the **refundable** `deposit` is charged only as an **opportunity cost** (you get it back, but the cash is tied up): `DEPOSIT_OPP_RATE_MONTHLY = 0.005` (≈ 6% p.a. ÷ 12). It is **not** fully amortized — that would overstate cost.
- **Parameters (fixed for now; revisit per run):** `STAY_MONTHS = 12`; deposit opportunity rate `6% p.a.`
- **Missing inputs:** a blank cost field is summed as `0` — but **absence ≠ zero** (99acres hides maintenance/brokerage; Housing's move-in needs the price-breakup popover). So the value can be a **lower bound**, recorded in the companion column **`calc_cost_basis`**: `full` (every cost field the *site* exposes was captured) or `lower-bound` (≥1 cost input absent/uncaptured → understates true cost). `rent` is required; if `rent` is blank, leave both columns empty.
- **Comparability:** compare `full` rows to `full` rows; a `lower-bound` value is a floor, not the real cost. Per-site cost-field map in `docs/processing-rules.md`.
- **First applied:** the trial round (`data/2026-06-18-trial-*-blr-1bhk/`).

### `calc_amenity_<name>` (on-demand amenity flags)
The settled representation for amenities (see `docs/data-schema.md`): capture stays a single raw `amenities` list; amenity-level access is **derived here** as boolean flags, **only for the amenities a run actually filters or scores on** — not one column per possible amenity.

- **Output:** one `calc_amenity_<name>` boolean column per amenity in use, e.g. `calc_amenity_gated`, `calc_amenity_lift`, `calc_amenity_power_backup`, `calc_amenity_security`.
- **Input:** the captured `amenities` string only (never re-browse the site).
- **Method:** match the row's `amenities` tokens against the amenity's synonym set in the **amenity-normalization map** (`docs/processing-rules.md` §amenities) — case-insensitive, substring-aware, and handling NoBroker's `key: value` form (e.g. `Gated Security: No` → gated = **false**, not true). `true` when a synonym is present and not negated; `false` when absent or explicitly negated.
- **`false` ≠ unknown:** `false` means "not mentioned / negated", which is how listings express absence. Leave the cell **blank only when `amenities` itself was uncaptured** for that row.
- **Don't rewrite capture:** these are appended `calc_` columns; the raw `amenities` cell is untouched.
- **Scope per run:** decide the small set of amenities that matter for the run before materializing flags; don't auto-expand to every token seen (that recreates the sparse wide table this decision rejected).
- **Status:** representation **settled (2026-06-18)**; the specific amenity set per run is chosen when a run needs amenity filtering. Not yet applied to any run.

## Candidate calculations (not yet committed)

From earlier brainstorming — confirm before relying on any:
- **Commute / distance** — distance or travel time from each listing to key locations (office, metro, …), plus a derived score.
- **Weighted ranking score** — combine multiple factors into one score to rank/shortlist within a site.

## When a calculation is chosen

1. Define its exact formula and which captured columns it reads here, including how missing inputs are handled.
2. Add its `calc_`-prefixed output column(s) to the derived-columns section of `docs/data-schema.md`.
3. Apply it in the processing step (see `docs/workflow.md`), leaving the captured columns untouched.

Calculations are computed **per site, within that site's table** — sites are not merged.
