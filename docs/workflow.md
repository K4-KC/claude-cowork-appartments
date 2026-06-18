# Workflow: one apartment-search run

How a single search run flows end to end. This is the contract between Cowork (Claude in Chrome) and Claude Code (this repo).

## Roles

- **Cowork (Chrome extension):** opens each site, applies the filters, browses listings, extracts fields, and writes the raw CSV directly into `data/<run-id>/<site>.csv`.
- **Claude Code (here):** authors the recipes Cowork follows, then validates and processes the raw CSVs into per-site comparison tables. Never browses the sites.

## Steps

1. **Name the run.** Pick a run id — a date or short label, e.g. `2026-06-17-koramangala-2bhk`. All output for the run lives under `data/<run-id>/`.

2. **Set the filters.** Fill in the search parameters from `docs/search-config.md` (city, localities, budget, BHK, …). They are the same for every site in the run.

3. **Capture per site.** How Cowork is handed a site depends on whether that site already has a recipe. Either way Cowork runs **inside this repo folder**, so it reads the referenced files directly.

   - **Normal run (recipe exists — the steady state).** Point Cowork at the site recipe `docs/sites/<site>.md` and give it the run id + filters (from `data/<run-id>/BRIEF.md`, or inline). The recipe is **self-contained**: its "Cowork — start here" header tells Cowork what else to auto-load (`docs/cowork-run.md` → `docs/data-schema.md` → `docs/rules.md`). **No kickoff message is generated.** Cowork applies the filters, walks the results, writes `data/<run-id>/<site>.csv` following `docs/data-schema.md`, and notes any drift from the recipe in `data/<run-id>/findings.md`.
   - **First-contact trial (no recipe yet — the design phase).** Use `docs/trial-protocol.md`. Here Claude Code *does* generate a copy-pastable **kickoff message** at `tmp/cowork-kickoff-<run-id>.txt` (transient, git-ignored), because there is no recipe to point Cowork at yet. Cowork discovers the capture method and records it in `data/<run-id>/findings.md`; Claude Code then distills the durable parts into `docs/sites/<site>.md` — after which future runs of that site are kickoff-free.

   `data/<run-id>/findings.md` is the agreed Cowork↔Claude Code channel for both trial and normal runs (raw findings on a trial; drift/confirmation on a normal run).

4. **Validate (here).** Load each raw CSV and check it against the schema: required columns present, units consistent, malformed rows flagged. Do not silently "fix" values — flag and ask.

5. **Calculate (here).** Apply the additional logic from `docs/calculations.md` (once defined) to each row, appending the derived `calc_` columns. This adds derived data alongside the captured columns — it never edits a captured value.

6. **Produce comparison tables.** Output one comparison table per site, kept distinct. Final format/destination is TBD — confirm with the user.

## Handoff notes

- Cowork writes raw files directly; this repo does not browse. If you find yourself wanting to open a site here, write/extend a recipe instead.
- The kickoff message in `tmp/` is a **design-time** convenience (first-contact trials only) and is git-ignored; normal runs need none — Cowork self-serves from `docs/sites/<site>.md` + `docs/cowork-run.md`. The durable artifacts of a run are the brief, the recipe, the captured CSV, and `data/<run-id>/findings.md`.
- Capture and recipe-building are iterative: when a trick works (or breaks) on a site, record it in that site's recipe doc so the next run is faster.
- If you want to keep an untouched raw capture alongside the processed table, agree on a naming convention first and document it in `docs/data-schema.md` — that lets a run be re-processed without re-browsing.
