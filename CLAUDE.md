# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This repo is the **design + processing hub** for an apartment-search workflow over Indian rental sites (housing.com, 99acres.com, nobroker.in, magicbricks.com). It does **not** open or browse those sites. Two different Claudes are involved:

- **Cowork (Claude in the Chrome extension)** — opens the sites, applies filters, surfs listings, and **writes the raw captured data directly into this repo** as CSV under `data/`.
- **Claude Code (here)** — designs the per-site browsing/extraction recipes Cowork follows, then cleans, validates, and runs calculations on the raw CSVs, producing per-site comparison tables.

Keep this split in mind: when working here you are the *designer and processor*, not the browser. If a task needs live browsing, that belongs to Cowork — write or extend a recipe for it instead of trying to fetch pages yourself.

## Goal

List apartments matching a set of filters across the four sites, run additional logic/calculations on them, and produce a **per-site comparison table** for each site. Sites are kept distinct — cross-site merge/dedup is not required unless the user asks.

## Workflow at a glance

A single search "run" flows like this (full detail in `docs/workflow.md`):

1. Define the run's filters/search parameters — see `docs/search-config.md`.
2. For each site, hand Cowork the per-site recipe (`docs/sites/<site>.md`) plus the filters; Cowork browses and writes `data/<run-id>/<site>.csv` (raw).
3. Here in Claude Code, clean/validate the raw CSVs against the schema (`docs/data-schema.md`).
4. Apply the additional calculations (`docs/calculations.md`) and produce the per-site comparison tables.

Per-site recipes are discovered by **trial and testing** — expect to iterate, and record what works in each site's doc as you learn it.

**Cowork handoff.** Cowork runs **inside this repo folder**, so it reads every file directly by repo-relative path. **Normal runs are kickoff-free:** point Cowork at the site recipe `docs/sites/<site>.md` (its "Cowork — start here" header auto-loads `docs/cowork-run.md` → `docs/data-schema.md` → `docs/rules.md`) plus the run id + filters, and it captures `data/<run-id>/<site>.csv`. **Only first-contact trials use a kickoff:** when a site has no recipe yet, Claude Code generates a copy-pastable **kickoff message** at `tmp/cowork-kickoff-<run-id>.txt` (transient, git-ignored) pointing Cowork at `docs/trial-protocol.md` + the filters; the recipe the trial produces is what makes future runs of that site kickoff-free. Either way, Cowork records what it learns into the run's **shared findings doc** `data/<run-id>/findings.md` (the agreed Cowork↔Claude Code channel); Claude Code distills the durable learnings into `docs/sites/<site>.md` and validates the captured CSV. Full detail in `docs/workflow.md` and `docs/cowork-run.md`.

## Repository layout

- `docs/` — modular documentation, one concern per file (see Documentation map below).
- `data/<run-id>/<site>.csv` — captured + processed listings. One folder per run, one CSV per site. See `data/README.md`.

## Data conventions

- Format is **CSV** — one file per site per run. Schema lives in `docs/data-schema.md`.
- The **captured (direct-from-site) column set is decided** — see `docs/data-schema.md`. Cowork writes those columns; derived `calc_` columns are computed here and remain TBD (`docs/calculations.md`).
- Runs are isolated under `data/<run-id>/` so search history is kept; do not overwrite a prior run's folder.
- Sites stay separate; do not merge listings across sites by default.
- Do not invent or fill in listing values. If Cowork could not capture a field, leave it blank — see the schema doc for the convention. Treat captured data as source-of-truth: validate and flag anything that looks off rather than silently "correcting" it.

## Committing

Committing should be quick and automatic — when work reaches a sensible point, stage and commit it directly (on the current branch) with a clear message, without pausing to ask. Push only when the user asks.

The **`/commit` skill** (`.claude/skills/commit/SKILL.md`) packages this into a one-shot, fully automatic command. It commits **only the files changed during the current conversation session** — never the repo's other pending changes — by staging an explicit, verified list of paths (never `git add -A`/`git add .`/`git commit -a`). It asks nothing and does not push. Use it whenever you want to commit this session's work.

## Documentation map

| File | Purpose |
|---|---|
| `docs/workflow.md` | End-to-end run lifecycle and the Cowork↔repo handoff |
| `docs/cowork-run.md` | Cowork's standing instructions for a **normal run** — the auto-load read order and capture steps; no kickoff needed |
| `docs/trial-protocol.md` | First-contact trial method (design-time, uses a kickoff): how to discover a site's fastest capture path and what findings to record |
| `docs/search-config.md` | The filter/search-parameter template for a run (the "set of filters") |
| `docs/data-schema.md` | CSV columns, units, and raw-vs-clean conventions |
| `docs/calculations.md` | The additional logic/calculations (`price_per_sqft` + `true_monthly_cost` defined; commute/ranking open) |
| `docs/processing-rules.md` | Cross-site validation/normalization rules the trials surfaced (area-basis, per-site cost-field map, date/token handling) |
| `docs/rules.md` | Operating rules and etiquette for capture + processing |
| `docs/sites/<site>.md` | Per-site browsing/extraction recipe — the self-contained run entry point (has a "Cowork — start here" header), refined via trial & testing |

## Open decisions (not yet settled)

These are intentionally unspecified; confirm with the user before relying on them, and update the relevant doc (not just this file) once decided so the knowledge stays modular:

- **Which calculations** to run per listing. **Decided:** `calc_price_per_sqft`, `calc_true_monthly_cost` (+ `calc_cost_basis` flag). **Still open:** commute/distance score and weighted ranking. See `docs/calculations.md`.
- **Amenities representation** — keep all amenities in one semicolon-separated `amenities` column, or split into one column per amenity. Provisional: single column. Detail in `docs/data-schema.md`.
- **Exact filter set and target city/localities** for a run. Template in `docs/search-config.md`.
- **Per-site recipes** — to be built and validated through trial runs.
- **Final delivery format** of the per-site comparison tables (Markdown, CSV, etc.).
