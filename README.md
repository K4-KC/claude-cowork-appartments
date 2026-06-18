# claude-cowork-appartments

A workflow for listing apartments matching a set of filters across Indian rental sites — housing.com, 99acres.com, nobroker.in, magicbricks.com — and running additional logic/calculations on them.

Two Claudes collaborate:

- **Cowork (Claude in the Chrome extension)** opens the sites, applies filters, browses, and writes the captured listings directly into this repo as CSV.
- **Claude Code (this repo)** designs the per-site browsing recipes, then validates, calculates on, and turns the data into per-site comparison tables. It does not browse the sites itself.

## Where things live

- `CLAUDE.md` — guidance and the big-picture architecture for working in this repo.
- `docs/` — modular docs: [`workflow.md`](docs/workflow.md), [`cowork-run.md`](docs/cowork-run.md), [`trial-protocol.md`](docs/trial-protocol.md), [`search-config.md`](docs/search-config.md), [`data-schema.md`](docs/data-schema.md), [`calculations.md`](docs/calculations.md), [`processing-rules.md`](docs/processing-rules.md), [`rules.md`](docs/rules.md), and per-site recipes under [`docs/sites/`](docs/sites/). (See the Documentation map in `CLAUDE.md` for the canonical list.)
- `data/<run-id>/<site>.csv` — captured + processed listings, one folder per run. See [`data/README.md`](data/README.md).

This project is being designed iteratively via trial and testing; the core per-listing calculations are defined, while commute/ranking calculations and the exact filters are still being decided.
