# Recipe: Magicbricks (magicbricks.com)

How Cowork should browse and extract from Magicbricks. **To be filled in via trial and testing** — record what works, what breaks, and the exact UI steps as you learn them.

## Trial status

- **Status:** First-contact trial in progress. Method to follow: `docs/trial-protocol.md`.
- **Active trial run:** `data/2026-06-18-trial-magicbricks-blr-1bhk/` (brief + findings template + sample CSV).
- Reference recipes: `docs/sites/housing.md`, `docs/sites/99acres.md`, `docs/sites/nobroker.md` (same filters — note where Magicbricks differs, especially login modals, deep-linkability, and cost-field coverage).
- Fill the sections below from the trial's findings; the TBDs are what the trial exists to answer.

## URL / entry point

- Base search URL: _TBD_
- Does it support filters in the URL (deep-linkable searches)? _TBD_

## Applying the run's filters

Map each filter from `docs/search-config.md` to this site's UI controls:

| Filter | How to set on this site | Notes |
|---|---|---|
| City | _TBD_ | |
| Locality | _TBD_ | |
| Budget | _TBD_ | |
| BHK | _TBD_ | |
| … | _TBD_ | |

## Walking results

- Pagination / infinite scroll behavior: _TBD_
- How to open a listing for full details: _TBD_

## Extraction → CSV

Cowork captures only the **captured columns** in `docs/data-schema.md` directly from the page. The derived `calc_` columns are added later by Claude Code in processing — never produce them here.

- Which captured columns (`docs/data-schema.md`) this site exposes, and where each lives on the page: _TBD_
- Fields this site does NOT provide: _TBD_

## Gotchas

- Login walls, popups, rate limits, layout quirks: _TBD_

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| _2026-06-18_ | _Trial set up; awaiting Cowork's first pass._ | _Run `data/2026-06-18-trial-magicbricks-blr-1bhk/` per `docs/trial-protocol.md`._ |
