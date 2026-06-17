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
- Cowork (Claude in Chrome) writes these files directly. Runs are not overwritten — each run gets a fresh folder so search history is kept.
- Sites are kept distinct — no combined/merged file by default.
- Optionally keep the run's filter values next to the data (see `../docs/search-config.md`) so the run is reproducible.
