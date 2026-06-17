# Additional logic / calculations

These produce **derived data** — values computed here in Claude Code from the columns Cowork captured, not anything read off the sites. Each calculation's output is a derived column and follows the derived-column rules in `docs/data-schema.md`: name it with a `calc_` prefix, append it after the captured columns, and never overwrite a captured value to hold the result.

**Status: mostly undecided.** The user chooses which calculations to run; do not assume any *candidate* below without confirming. **One calculation is now defined** (`calc_price_per_sqft`), introduced during the Housing.com trial.

## Defined calculations

### `calc_price_per_sqft`
- **Formula:** `rent ÷ area` (both captured columns), rounded to 2 decimals.
- **Unit:** ₹ per sq.ft per **month** (rent is monthly).
- **Basis:** uses the captured `area` on its captured `area_basis`. Compare like-for-like (e.g. all built-up); built-up and carpet ₹/sq.ft are **not** directly comparable, so flag when a table mixes bases.
- **Missing inputs:** if `rent` or `area` is blank/zero, leave `calc_price_per_sqft` empty — don't guess.
- **First applied:** Housing.com trial `data/2026-06-18-trial-housing-blr-1bhk/`.

## Candidate calculations (not yet committed)

From earlier brainstorming — confirm before relying on any:

- **True monthly cost** — rent + maintenance + amortized deposit / brokerage / one-time fees spread over the expected stay duration.
- **Commute / distance** — distance or travel time from each listing to key locations (office, metro, …), plus a derived score.
- **Weighted ranking score** — combine multiple factors into one score to rank/shortlist within a site.

## When a calculation is chosen

1. Define its exact formula and which captured columns it reads here, including how missing inputs are handled.
2. Add its `calc_`-prefixed output column(s) to the derived-columns section of `docs/data-schema.md`.
3. Apply it in the processing step (see `docs/workflow.md`), leaving the captured columns untouched.

Calculations are computed **per site, within that site's table** — sites are not merged.
