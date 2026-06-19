# Run brief — Google Maps geo-enrichment (full run, resumable)

- **Run id:** `2026-06-19-geo-blr-1bhk-furnished-bachelors`
- **Source:** Google Maps — **normal run** (recipe exists: `docs/sites/google-maps.md`). **No kickoff** needed.
- **Driver:** Cowork (Claude in Chrome). Drift/findings → `findings.md` in this folder.
- **Goal:** geo-enrich the 728-listing milestone (99acres 350 + housing 328 + nobroker 50), deduped to **590 unique buildings**.

## Read order (auto-load)
`docs/cowork-run.md` → **`docs/sites/google-maps.md`** (the method + speed strategy + gate) → `docs/data-schema.md` (*Geo companion file* columns) → `docs/rules.md`.

## What to do
1. Work **`buildings-worklist.csv`** (in this folder) **top to bottom** — it's ordered building-level first, then address, then locality-only, so the highest-confidence buildings are done first. Each row carries the `geo_key`, the `geocode_query_hint`, and the source listing.
2. For each building, follow `docs/sites/google-maps.md`: **geocode + set the 3-tier `geo_confidence` gate**, then (if `building`/`locality`) capture the metric set via the directions deep-links.
3. **Write that building's row to `geo.csv` IMMEDIATELY — one at a time, never buffered.** Carry the worklist's `geo_key` into the row. `geo.csv` already has the locked 48-column header.

## Resumability (the whole point)
`geo.csv` is the **source of truth and resume point**. On any session: **skip every `geo_key` already present in `geo.csv`** and continue down the worklist. Stop/resume freely — zero re-work. Mark `done` in the worklist if convenient, but `geo.csv` presence is authoritative.

## Decisions baked in (2026-06-19)
- **Car route = Maps "Best route"** (top/first-listed), uniformly.
- **`supermarket` only** — no separate `grocery` (Maps can't distinguish them).
- **`*_mm_min` = transit time as shown** (bus or metro, whichever is fastest — NOT pure metro). **`*_mm_km` is always blank** (Maps transit shows no distance).
- **Metro** = nearest station on **Green / Purple / Yellow** only.
- **Instamart** = two-wheeler to nearest dark store; **blank in outer localities** (none present).

## Speed (see recipe → "Speed strategy")
Harvest each mode's **time** off the mode bar (one load shows all modes' time; only distance is per-mode → transit & metro-car time are free); route fixed destinations by **cached coords** (anchors below; metro via the station table once built); **deep-link, never click**; `browser_batch` the navigate→wait→read. Cached anchors: Indiranagar Metro `12.9782619,77.6385257` · M.G. Road Metro `12.9756295,77.6066227` · Koramangala/Nexus Mall `12.934692,77.6111212`.

## Ground rules
Browse like a careful human; ~3 s between loads (no throttle seen at that pace). Don't bypass logins/CAPTCHAs or script the Maps API. **Never invent** a coordinate/distance/time — blank is correct when Maps won't give it. `none`-tier buildings: lat/lng + all distances blank.
