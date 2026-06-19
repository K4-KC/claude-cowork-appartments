# Findings — geo run `2026-06-19-geo-blr-1bhk-furnished-bachelors`

Handback for the full Google Maps geo-enrichment run (590-building worklist, resumable). Cowork records progress + any drift here; Claude Code refreshes `docs/sites/google-maps.md` from it.

- **Driver:** Cowork · **Status:** _in progress_ · **Last `geo_key` written:** `B|vasantha homes|btm layout`

## Progress log
_(append per session: date · buildings done this session · cumulative count in `geo.csv` · where you stopped)_

- **2026-06-19 (session 1):** 3 buildings captured → cumulative **3 / 590** in `geo.csv`. Worklist rows 1–3 (`godrej royale woods|devanahalli`, `btm|btm 2nd stage`, `sowparnika ashiyana|whitefield`) done + marked `done` in worklist. Stopped at worklist row 4 (`colive franny|koramangala`) — **587 remaining**. All metrics validated (48-col shape, no dup keys, units normalized, gate applied). Resume = next not-`done` worklist row whose `geo_key` is absent from `geo.csv`.
- **2026-06-19 (session 2):** +3 buildings (rows 4–6: `colive franny|koramangala`, `sowparnika unnathi|attibele`, `sowparnika indraprastha|soukya road`) → cumulative **6 / 590**. **584 remaining.** Switched to all-text capture (no screenshots): geocode via `/maps/place` URL for coords + `get_page_text` for type; anchors = drive-load (`!3e0`) for car km/min + transit-load (`!3e3`) for `*_mm_min` (first-listed = Maps default), no mode-bar screenshot; POI walk+drive batched text-only. Faster + lighter. **Standardized `*_mm_min` = first-listed (Maps default/best) transit option**, consistent with the session-1 mode-bar reads (not the absolute-min, which can be an anomalous future express). Hit the wrong-region trap once (`Apollo Pharmacy Koramangala 6th Block` → Mysore) → switched to uniquely-named nearby POIs; generic names (`Apollo Pharmacy`, `Everyday Store`) are the risk, so prefer a uniquely-named result when the first is a common chain.
- **2026-06-19 (session 3):** +3 buildings (rows 7–9: `mantri classic|st bed`, `lalitha homes|marathahalli`, `vasantha homes|btm layout`) → cumulative **9 / 590**. **581 remaining.** Method stable (all-text). Per-building cost ~13–15 loads; this is inherently a many-session run. **Git note:** session-1 commit landed (`a191633`); sessions 2–3 changes are saved to disk + staged but a stale `.git/HEAD.lock` (host-held, sandbox can't unlink) blocks new commits — run `git commit` host-side to finalize.

## Drift / issues to feed back into the recipe
_(URL/UI changes, throttling, a metric that stopped working, gate edge-cases, worklist rows that wouldn't geocode, etc.)_

- **Geocode coords often absent from the `/maps/search/` URL.** For both building- and locality-tier resolves, `/maps/search/<query>` frequently stayed a search URL with **no `!3d…!4d…`**. Workaround that worked every time: re-navigate to `https://www.google.com/maps/place/<name>,<area>` and read `!3d<lat>!4d<lng>` from the resolved URL. (Building 1 search URL *did* carry coords; buildings 2 & 3 did not.) Recipe should make the place-URL step the default coord source.
- **Don't batch multiple screenshot-bearing directions loads in one `browser_batch`.** Batching all 3 anchors in one call hit a transient `get_page_text` "No text content found" on the 2nd load, which **aborted the batch AND omitted the screenshots already captured** — total loss of that round-trip. Fix: one `browser_batch` per screenshot-bearing load (geocode, each anchor, metro-walk), with `wait≥4s`. Confirmed reliable after switching.
- **POI walk+drive CAN be batched in one round-trip (text-only, no screenshot).** Same destination, two navigates (`!3e2` then `!3e0`) + `get_page_text` each. Read walk km/min from the walk card and car km/min from the drive card — no mode-bar screenshot needed for POIs. Big speed/clarity win; only anchors + metro still need a screenshot (for transit / metro-car time off the mode bar).
- **Instamart relevance ≠ nearest.** Building 3's first result (Marathahalli AECS, ~10 km) was not the nearest; the actual nearest dark store (Gandhipura, 5.2 km) was lower in the list. Pick the Instamart pin by geographic nearness (plus-code/area), not list order. Same caveat applies more mildly to the daily-needs POIs (relevance puts big-brand names first).
- **Capture hour ≈ 18:50 IST (evening rush)** for this session — the trial's were ~08:00 (morning rush). Both inflate `*_min`; per recipe lever 6, rank on **distance** (traffic-invariant) and treat times as approximate, or fix a capture hour before mixing sessions.
- **No throttle/CAPTCHA** across ~45 loads at ~3–4 s spacing on a signed-in session. Wrong-region trap is real (account has Mysuru/Ooty saved places) but every Bengaluru query resolved correctly; kept verifying resolved titles.

## Per-building skips / blanks worth noting

- **`B|godrej royale woods|devanahalli`** (building) — Devanahalli, ~35 km N near airport. Metro impractical (nearest allowed-line = K.R. Pura/Purple, 31.7 km walk). **Instamart blank** — no dark store within range (nearest ~28 km).
- **`B|btm|btm 2nd stage`** (locality) — `building_name` "btm" was junk → fell back to BTM Layout 2nd Stage **area centroid**; all metrics locality-level/approximate.
- **`B|sowparnika ashiyana|whitefield`** (building) — resolves to Samethanahalli (NE of Whitefield toward Hoskote), further out than the "Whitefield" label; metro = Whitefield (Kadugodi)/Purple terminus ~7 km (not walkable).
- `metro_car_km` left **blank** for all rows by design (recipe lever-2 demotion); `*_mm_km` blank by schema (transit shows no distance).
- **`B|sowparnika unnathi|attibele`** (building) — far SE outer town (~30 km). **Instamart blank** (nearest dark store ~15 km+ at Sarjapur/Mullur). Metro Delta Electronics Bommasandra/Yellow ~12 km.
- **`B|sowparnika indraprastha|soukya road`** (building) — east Hoskote fringe. Instamart Gandhipura **8.1 km** (recorded but likely beyond practical serving range). Metro Whitefield Kadugodi/Purple ~9 km.
