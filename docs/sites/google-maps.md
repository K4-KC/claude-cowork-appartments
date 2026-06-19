# Recipe: Google Maps (geo-enrichment source)

> **Cowork — start here.** This is **not** a rental-listing site — it is the **geo-enrichment / derived-run** source. You geocode each building once and read distance/time metrics off Maps into a per-building `geo.csv`. This run is triggered by **"run a derived run for `<run-id>`"** (see `docs/cowork-run.md` → *Run types*). **Read order:** the run's **`BRIEF.md`** → this file → its **`buildings-worklist.csv`** → `docs/data-schema.md` (the *geo* columns) → `docs/rules.md`. The run is driven by that **`buildings-worklist.csv`** — a deduped unique-building list (ordered high-confidence first) that Claude Code prepared (not a site search) — so it runs **after** the listing CSVs are captured. No kickoff needed once this recipe exists — kickoffs are only for the first-contact trial (`docs/trial-protocol.md`).

Distilled from the first-contact trial `data/2026-06-19-trial-maps-geo-blr-1bhk/` (2026-06-19, 3 buildings, all 3 geocode tiers validated). If live Maps has drifted, record it in the run's `findings.md`.

## What this produces

One row **per unique building** in `geo.csv` (schema in `docs/data-schema.md` → *Geo companion file*): a geocode (`lat`,`lng`,`geo_confidence`) plus, for that point, nearest-metro / downtown-anchor / daily-needs-POI / Instamart distances and times. Buildings are **deduped across all sites first** (a society geocoded once serves every listing in it, on every site) and **cached across runs** — a building already in the master geo store is never re-captured.

---

## The worklist — Cowork's input (one row per building)

Process **`data/<run-id>/buildings-worklist.csv`** top to bottom (it's ordered building-level → address → locality-only, so the most reliable buildings come first). Each row's columns:

| Column | Use |
|---|---|
| `geo_key` | The building's unique id. **Write it verbatim into the `geo_key` column of `geo.csv`** — it is also the **resume key** (skip keys already in `geo.csv`). |
| `geocode_query_hint` | **The search string to geocode** — use it as-is in Maps (it's the prepared `building_name, locality, Bengaluru`, or the address). Prefer it over re-assembling your own query. |
| `locator_type` | `building` / `address` / `locality` — a *hint* about the likely tier. **Do not** copy it into `geo.csv`, and **do not** treat it as the confidence: you still derive `geo_confidence` yourself from the resolved Maps place type (the gate below). |
| `building_name`, `locality`, `address` | Provenance + fallback context for the geocode. |
| `geocode_from_site`, `geocode_from_listing_id` | The listing this building came from → write into `geo.csv`'s `source_site` / `source_listing_id`. |
| `n_listings`, `sites`, `done` | Bookkeeping. `done` is optional convenience; **`geo.csv` presence is the authoritative resume signal**, not `done`. |

---

## ⚡ Speed strategy (read this first — geo capture is the slow step)

Measured steady-state in the trial: **~6–7 min/building** for the full metric set (~24 directions page-loads + ~6 category searches). The cost driver is **page-loads**, because *distance* for a destination only shows for the currently-selected travel mode. The levers below cut both the loads per building and the number of buildings. Apply them in order — the first two are multiplicative.

1. **Dedup to unique buildings, then cache across runs (biggest lever — cuts the building *count*).** Claude Code dedups the run's listings to unique `(building, locality)` before handing Cowork a worklist, and skips any building already in the master geo store. 728 listings collapse to far fewer unique buildings; re-runs add only new ones. *(If the user also scopes to a post-filter shortlist, that multiplies on top — the cheapest 10×.)*
2. **Harvest TIME off the mode bar — don't reload for it (cuts loads/building).** One directions load shows **every mode's *time* at once** (drive / two-wheeler / transit / walk) in the top mode bar; only *distance* is per-mode. So any metric where we only need **time** costs **zero extra loads** — read it from the mode bar of a load you already did to that destination:
   - **Transit (`mm`)** has no obtainable distance anyway → it is **time-only** → read `*_mm_min` off the **car** load's mode bar to the same anchor. The 3 anchors drop from **6 loads → 3**.
   - **Metro by car**: driving *distance* to a nearby metro is low-value → demote `metro_car_*` toward time-only → read `metro_car_min` off the **walk** load's mode bar. Metro drops from **2 loads → 1**.
3. **Cache fixed-destination coordinates — eliminate destination searches.** Route by raw `lat,lng`, never a re-typed place name (this also dodges the wrong-region trap — see Gotchas):
   - **Downtown anchors (fixed, cached):** Indiranagar Metro ≈ `12.9782619,77.6385257` · M.G. Road Metro ≈ `12.9756295,77.6066227` · Koramangala (now **Nexus Mall**, ex-Forum) ≈ `12.934692,77.6111212`.
   - **Metro stations (build-out lever):** the operational **Green / Purple / Yellow** stations are a finite fixed set. Once a `station,line,lat,lng` reference table exists, Claude Code picks the nearest allowed-line station by straight-line distance and hands Cowork the station's coords — removing the per-building "metro station" search and **guaranteeing the line constraint**. Until then, use the category search below.
4. **Deep-link everything; never click.** Navigate directly to `/maps/place/…` and `/maps/dir/…` URLs — clicking result cards is flaky and slow. Claude Code can pre-generate the whole per-building directions-URL batch once Cowork reports the geocoded `lat,lng` (fixed-destination URLs are fully templated; only the origin varies).
5. **`browser_batch` the `navigate → wait ~3s → get_page_text`** (and two modes back-to-back) per call — this is what holds it to ~6 min. Read route cards with `get_page_text`; a screenshot is only needed to read the mode bar.
6. **Rank on distance, standardize the clock.** Distance is traffic-invariant; the trial's times were 08:00-IST rush-inflated. Lean on distance for ranking and treat time as approximate (or fix a capture hour) so data needn't be re-captured.

**Net projection:** levers 2–3 take a full building from ~24 loads to **~15 loads + ~6 searches (~4 min)**; merging supermarket/grocery and demoting a POI mode to time-only can reach ~10–12 loads. Lever 1 then cuts how many buildings you pay that for.

---

## Geocode + the 3-tier confidence gate (do first, per building)

Search `building_name + locality + Bengaluru`; if the name is junk/blank, search the **address**; last resort, the **locality**. Read coords from the resolved `…/maps/place/…!3d<lat>!4d<lng>` URL. Then set `geo_confidence` by the resolved place's **type header** — it gates everything downstream:

- **`building`** — resolves to an *"Apartment building"*, a named society, or a street address with a house number, in the right area of Bengaluru. Record `lat`,`lng`; capture all metrics. *(A missing `building_name` does NOT force a downgrade — trial #3 had no name but its address resolved straight to the named society.)*
- **`locality`** — only a *"Town"* / neighbourhood / area resolves (junk or missing locator). Record the **centroid** `lat`,`lng`, set `locality`, and **note metrics are approximate** in `geo_notes`.
- **`none`** — can't place even the locality, or the only match is the wrong city/region. Leave `lat`,`lng` **and all distances blank**. Never guess a pin.

## Capture method (directions deep-link)

```
https://www.google.com/maps/dir/<lat,lng>/<destination>/data=!4m2!4m1!3e<MODE>
```
- **Origin** = the building's raw `lat,lng` (pins the exact start; no re-geocoding).
- **`<destination>`** = a cached `lat,lng` (anchors, metro once the table exists) or a category search for the dynamic POIs.
- **Mode token `!3e<MODE>`:** `0`=drive · `2`=walk · `3`=transit · `9`=two-wheeler · (`1`=bicycle).
- **Set the mode in the URL** — clicking a mode in the panel does **not** refresh the route list in an automated session (mode-bar highlight changes but the cards stay stale). **One load per mode you need *distance* for.** Time for the other modes comes free from the mode bar (lever 2).

## Incremental persistence & resume (required)

The full run is long (many buildings) and spans context compaction, so **persist each building the moment it's done — never buffer in context** (user constraint, 2026-06-19):

- Capture **one building end-to-end, then immediately append its row to `geo.csv`** before starting the next. Do **not** hold several buildings' results in working memory to write in a batch — a compaction would lose them.
- The on-disk `geo.csv` is the **source of truth and the resume point**. **First action of every session: read `geo.csv`, collect the set of `geo_key` values already present, and skip those worklist rows** — then continue down the worklist. Match on **`geo_key`** (the column both `geo.csv` and the worklist share) — *not* `calc_geo_key`, which is a listing-side join key that does **not** exist in either file.
- **Before appending a row, confirm its `geo_key` is not already in `geo.csv`.** The file has no enforced uniqueness, so a missed skip would silently duplicate a building (and fan out the downstream join). This is the same dedup/cache that powers speed lever 1.
- One row per building, written in **worklist order**, so progress is always inspectable and the run can stop/resume at any point with zero re-work. Time is acceptable *as long as this holds*.

## Destinations & their reads

| Group | Columns | How |
|---|---|---|
| **Nearest metro** (Green/Purple/Yellow only) | `metro_name`, `metro_line`, `metro_walk_km/min`, `metro_car_km/min` | Nearest allowed-line station (cached table → Claude Code picks; else category-search `metro station`, distance-ordered). Station place page shows **"Metro services: <X> Line"** — use it to enforce the line. Load **walk** (dist+time); take car time off the mode bar (demote car distance, lever 2). |
| **Indiranagar / MG Road / Koramangala** | `<area>_car_km/min`, `<area>_mm_min` | Route to the **cached anchor coords**. Load **drive** (dist+time); read **transit** time off the same load's mode bar. `*_mm_km` is **not obtainable** (transit panel shows no distance) → leave blank. |
| **Daily-needs POIs**: hospital, pharmacy, gym, supermarket | `<cat>_name`, `<cat>_walk_km/min`, `<cat>_car_km/min` | Category search near the pin (relevance ≈ nearest; take first sensible). Load walk + drive. **`supermarket` is the single daily-shopping POI** — `grocery_*` was dropped because Maps returns the same ranked list for both (decision below). |
| **Instamart** | `instamart_name`, `instamart_2w_km/min` | Search "Swiggy Instamart" (listed as *"Grocery delivery service"*); nearest pin, **two-wheeler** mode. Dark stores **exist in the city** but are several km away and **absent in outer towns** → blank there. |

## Gotchas (also feed `docs/processing-rules.md` for read/normalize)

- **Verify every resolved place, including POI destinations.** A `MedPlus … Nelamangala, BH Road, Bengaluru` query silently resolved to a MedPlus near **Mysore (~150 km)** and returned an empty route (signed-in account had Mysore saved places; ambiguous tokens + trailing "Bengaluru" pulled it). Prefer a uniquely-named result + town name, **drop the trailing "Bengaluru"**, or route by the POI's own coords. Trust the resolved title/coords, never the raw query.
- **`mm` is transit-as-shown — bus *or* metro, whichever is fastest — NOT pure walk+metro.** For short inner-city hops the fastest transit is usually a **bus**; metro only wins on long hauls. Flag this; pure-metro time isn't directly available from Maps.
- **Units/format:** distance is `km` ≥1 km but **metres** (`550 m`) below → normalize to km. Time: `33 min`, `1 hr 8 min`→68, `2 hr 24 min`→144 → integer minutes.
- **Route selection:** Maps' **"Best route"** is occasionally **not the shortest distance** (trial saw 1.6 km recommended vs 0.8 km alt). The trial recorded **top/first-listed** for car and **shortest duration** for transit — standardize this (Open decisions).
- **Car distance can exceed walk distance** on short hops (one-ways, highway detours) — not an error.
- **No CAPTCHA / rate-limit / login wall** across ~70+ loads at ~3 s spacing (signed-in). Keep the ~3 s spacing.

## Resolved decisions (2026-06-19)

1. **Route-selection** — use Maps' **"Best route"** (top/first-listed) for car, applied uniformly. Accepted that it's occasionally not the shortest distance. (Transit: shortest duration shown.)
2. **`supermarket` vs `grocery`** — **single `supermarket` POI**; `grocery_*` columns dropped (Maps returns the same ranked list for both, so the distinction isn't capturable).
3. **`mm`** — **keep** `*_mm_min` (transit time as shown — bus or metro, whichever is fastest). `*_mm_km` stays dropped (permanently blank).
4. **Scope** — **all unique buildings** (no shortlist). Time is acceptable **provided the run is incrementally persisted** (see *Incremental persistence & resume* above).
