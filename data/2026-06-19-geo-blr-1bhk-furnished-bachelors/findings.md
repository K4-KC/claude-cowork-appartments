# Findings — geo run `2026-06-19-geo-blr-1bhk-furnished-bachelors`

Handback for the full Google Maps geo-enrichment run (590-building worklist, resumable). Cowork records progress + any drift here; Claude Code refreshes `docs/sites/google-maps.md` from it.

- **Driver:** Cowork · **Status:** _in progress_ · **Last `geo_key` written:** `B|sowparnika ashiyana|whitefield`

## Progress log
_(append per session: date · buildings done this session · cumulative count in `geo.csv` · where you stopped)_

- **2026-06-19 (session 1):** 3 buildings captured → cumulative **3 / 590** in `geo.csv`. Worklist rows 1–3 (`godrej royale woods|devanahalli`, `btm|btm 2nd stage`, `sowparnika ashiyana|whitefield`) done + marked `done` in worklist. Stopped at worklist row 4 (`colive franny|koramangala`) — **587 remaining**. All metrics validated (48-col shape, no dup keys, units normalized, gate applied). Resume = next not-`done` worklist row whose `geo_key` is absent from `geo.csv`.

## Drift / issues to feed back into the recipe
_(URL/UI changes, throttling, a metric that stopped working, gate edge-cases, worklist rows that wouldn't geocode, etc.)_

- **Geocode coords often absent from the `/maps/search/` URL.** For both building- and locality-tier resolves, `/maps/search/<query>` frequently stayed a search URL with **no `!3d…!4d…`**. Workaround that worked every time: re-navigate to `https://www.google.com/maps/place/<name>,<area>` and read `!3d<lat>!4d<lng>` from the resolved URL. (Building 1 search URL *did* carry coords; buildings 2 & 3 did not.) Recipe should make the place-URL step the default coord source.
- **Don't batch multiple screenshot-bearing directions loads in one `browser_batch`.** Batching all 3 anchors in one call hit a transient `get_page_text` "No text content found" on the 2nd load, which **aborted the batch AND omitted the screenshots already captured** — total loss of that round-trip. Fix: one `browser_batch` per screenshot-bearing load (geocode, each anchor, metro-walk), with `wait≥4s`. Confirmed reliable after switching.
- **POI walk+drive CAN be batched in one round-trip (text-only, no screenshot).** Same destination, two navigates (`!3e2` then `!3e0`) + `get_page_text` each. Read walk km/min from the walk card and car km/min from the drive card — no mode-bar screenshot needed for POIs. Big speed/clarity win; only anchors + metro still need a screenshot (for transit / metro-car time off the mode bar).
- **Instamart relevance ≠ nearest.** Building 3's first result (Marathahalli AECS, ~10 km) was not the nearest; the actual nearest dark store (Gandhipura, 5.2 km) was lower in the list. Pick the Instamart pin by geographic nearness (plus-code/area), not list order. Same caveat applies more mildly to the daily-needs POIs (relevance puts big-brand names first).
- **No throttle/CAPTCHA** across ~45 loads at ~3–4 s spacing on a signed-in session. Wrong-region trap is real (account has Mysuru/Ooty saved places) but every Bengaluru query resolved correctly; kept verifying resolved titles.

## Per-building skips / blanks worth noting

- **`B|godrej royale woods|devanahalli`** (building) — Devanahalli, ~35 km N near airport. Metro impractical (nearest allowed-line = K.R. Pura/Purple, 31.7 km walk). **Instamart blank** — no dark store within range (nearest ~28 km).
- **`B|btm|btm 2nd stage`** (locality) — `building_name` "btm" was junk → fell back to BTM Layout 2nd Stage **area centroid**; all metrics locality-level/approximate.
- **`B|sowparnika ashiyana|whitefield`** (building) — resolves to Samethanahalli (NE of Whitefield toward Hoskote), further out than the "Whitefield" label; metro = Whitefield (Kadugodi)/Purple terminus ~7 km (not walkable).
- `metro_car_km` left **blank** for all three by design (recipe lever-2 demotion); `*_mm_km` blank by schema (transit shows no distance).

## Per-building skips / blanks worth noting
_(buildings that hit `none` tier, Instamart blanks, anything left blank that wasn't obvious)_

- 
