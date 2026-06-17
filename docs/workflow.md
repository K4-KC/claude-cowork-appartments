# Workflow: one apartment-search run

How a single search run flows end to end. This is the contract between Cowork (Claude in Chrome) and Claude Code (this repo).

## Roles

- **Cowork (Chrome extension):** opens each site, applies the filters, browses listings, extracts fields, and writes the raw CSV directly into `data/<run-id>/<site>.csv`.
- **Claude Code (here):** authors the recipes Cowork follows, then validates and processes the raw CSVs into per-site comparison tables. Never browses the sites.

## Steps

1. **Name the run.** Pick a run id — a date or short label, e.g. `2026-06-17-koramangala-2bhk`. All output for the run lives under `data/<run-id>/`.

2. **Set the filters.** Fill in the search parameters from `docs/search-config.md` (city, localities, budget, BHK, …). They are the same for every site in the run.

3. **Capture per site.** For each of the four sites, give Cowork:
   - the per-site recipe in `docs/sites/<site>.md`, and
   - the run's filters.

   Cowork applies the filters on the site, walks the results, and writes `data/<run-id>/<site>.csv` following `docs/data-schema.md`.

4. **Validate (here).** Load each raw CSV and check it against the schema: required columns present, units consistent, malformed rows flagged. Do not silently "fix" values — flag and ask.

5. **Calculate (here).** Apply the additional logic from `docs/calculations.md` (once defined) to each row, adding the computed columns.

6. **Produce comparison tables.** Output one comparison table per site, kept distinct. Final format/destination is TBD — confirm with the user.

## Handoff notes

- Cowork writes raw files directly; this repo does not browse. If you find yourself wanting to open a site here, write/extend a recipe instead.
- Capture and recipe-building are iterative: when a trick works (or breaks) on a site, record it in that site's recipe doc so the next run is faster.
- If you want to keep an untouched raw capture alongside the processed table, agree on a naming convention first and document it in `docs/data-schema.md` — that lets a run be re-processed without re-browsing.
