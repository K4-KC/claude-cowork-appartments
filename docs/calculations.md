# Additional logic / calculations

These produce **derived data** — values computed here in Claude Code from the columns Cowork captured, not anything read off the sites. Each calculation's output is a derived column and follows the derived-column rules in `docs/data-schema.md`: name it with a `calc_` prefix, append it after the captured columns, and never overwrite a captured value to hold the result.

**Status: not yet decided.** The user will choose which calculations to run; do not assume any of these without confirming.

Candidate calculations (from earlier brainstorming — none committed):

- **True monthly cost** — rent + maintenance + amortized deposit / brokerage / one-time fees spread over the expected stay duration.
- **Price per sq.ft** — rent (or maintenance-inclusive cost) ÷ area, with `area_basis` noted.
- **Commute / distance** — distance or travel time from each listing to key locations (office, metro, …), plus a derived score.
- **Weighted ranking score** — combine multiple factors into one score to rank/shortlist within a site.

## When a calculation is chosen

1. Define its exact formula and which captured columns it reads here, including how missing inputs are handled.
2. Add its `calc_`-prefixed output column(s) to the derived-columns section of `docs/data-schema.md`.
3. Apply it in the processing step (see `docs/workflow.md`), leaving the captured columns untouched.

Calculations are computed **per site, within that site's table** — sites are not merged.
