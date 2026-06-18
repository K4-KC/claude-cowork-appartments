# data/

Captured and processed listings, organized **by run, then by site**:

```
data/
  <run-id>/                 # e.g. 2026-06-17-koramangala-2bhk
    housing.csv
    99acres.csv
    nobroker.csv
    magicbricks.csv
    BRIEF.md                # run brief: filters + instructions handed to Cowork
    findings.md             # shared Cowork↔Claude Code handback doc for the run
```

- One folder per search run; one CSV per site. See `../docs/data-schema.md` for columns and conventions.
- Each CSV holds two kinds of columns: **captured** columns taken directly from the site by Cowork (source of truth, plain names), and **derived** columns computed here during post-processing (prefixed `calc_`, appended after the captured ones). See `../docs/data-schema.md` for the convention.
- Cowork (Claude in Chrome) writes the captured columns directly; Claude Code appends the `calc_` columns later. Runs are not overwritten — each run gets a fresh folder so search history is kept.
- Sites are kept distinct — no combined/merged file by default.
- Each run records its filter values in `BRIEF.md`; Cowork reads them from there on a normal (kickoff-free) run, and it keeps the run reproducible. See `../docs/search-config.md` and `../docs/cowork-run.md`.
- `findings.md` is the **shared handback doc** for a run: Cowork records what it learned while browsing (access walls, deep-link URL template, field coverage, pagination, quirks), and Claude Code distills the durable parts into `../docs/sites/<site>.md`. Defacto for trial and real runs. Kickoff messages (git-ignored `tmp/`) are a **design-time** aid used only for first-contact trials; **normal runs are self-served** from the site recipe — see `../docs/cowork-run.md`.
