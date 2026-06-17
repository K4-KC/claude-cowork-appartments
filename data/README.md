# data/

Captured and processed listings, organized **by run, then by site**:

```
data/
  <run-id>/                 # e.g. 2026-06-17-koramangala-2bhk
    housing.csv
    99acres.csv
    nobroker.csv
    magicbricks.csv
```

- One folder per search run; one CSV per site. See `../docs/data-schema.md` for columns and conventions.
- Each CSV holds two kinds of columns: **captured** columns taken directly from the site by Cowork (source of truth, plain names), and **derived** columns computed here during post-processing (prefixed `calc_`, appended after the captured ones). See `../docs/data-schema.md` for the convention.
- Cowork (Claude in Chrome) writes the captured columns directly; Claude Code appends the `calc_` columns later. Runs are not overwritten — each run gets a fresh folder so search history is kept.
- Sites are kept distinct — no combined/merged file by default.
- Optionally keep the run's filter values next to the data (see `../docs/search-config.md`) so the run is reproducible.
