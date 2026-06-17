# Additional logic / calculations

**Status: not yet decided.** The user will choose which calculations to run; do not assume any of these without confirming.

Candidate calculations (from earlier brainstorming — none committed):

- **True monthly cost** — rent + maintenance + amortized deposit / brokerage / one-time fees spread over the expected stay duration.
- **Price per sq.ft** — rent (or maintenance-inclusive cost) ÷ area, with `area_basis` noted.
- **Commute / distance** — distance or travel time from each listing to key locations (office, metro, …), plus a derived score.
- **Weighted ranking score** — combine multiple factors into one score to rank/shortlist within a site.

## When a calculation is chosen

1. Define its exact formula and inputs here, including how missing inputs are handled.
2. Add its output column(s) to `docs/data-schema.md`.
3. Apply it in the processing step (see `docs/workflow.md`).

Calculations are computed **per site, within that site's table** — sites are not merged.
