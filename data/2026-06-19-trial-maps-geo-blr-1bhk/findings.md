# Findings — Google Maps geo-enrichment trial `2026-06-19-trial-maps-geo-blr-1bhk`

First-contact trial of **Google Maps** as a geo source. Goal: the **method + feasibility**, not the data. Filled by Cowork (Claude in Chrome) while browsing; Claude Code distills the durable parts into `docs/sites/google-maps.md` and locks the geo schema.

- **Filled by:** Cowork · **Date:** 2026-06-19 · **Status:** 3 / 3 buildings captured (full metric set)

> All three rows are in `geo.csv`. Anything Maps wouldn't give is left **blank** there and explained below. Nothing was invented. Times were read at ~08:00 IST (the sandbox clock is UTC; Maps showed ~8 AM local departures), i.e. morning rush — see Q7.

## Outcome

- Buildings captured: **3 / 3**. Gate tier each landed in:
  - #1 **Srinivasa Nilaya** (Wilson Garden) → **building** (exact named-society pin).
  - #2 **"new house"** / Nelamangala → **locality** (junk name; fell back to Nelamangala Town centroid).
  - #3 **Ittina Sarva 1** / Bommanahalli (no building_name) → **building** (address resolved to the named society "Ittina Sarva 1 Apartments (ISOWA)").
- Things Maps would not give (left blank, all rows):
  - **Transit distance** (`*_mm_km` for indiranagar/mg_road/koramangala): the transit panel shows time, departure/arrival, fare and transfers but **no trip distance** → blank everywhere.
  - **Instamart for #2 Nelamangala**: no dark store in the town (nearest pin ~35 km away in the city) → blank.

## The 7 trial questions

### 1. Geocode reliability + the gate
All three paths resolved, one per tier:

- **Named building (#1):** searching `Srinivasa Nilaya Lakkasandra Extension Wilson Garden Bengaluru` returned two candidates; the first ("Srinivasa Nilaya", *Apartment building*, "12, 1st Cross") was the exact society. Its reverse-geocoded address (Arekempanahalli / Hombegowda Nagar 560011) is adjacent to the listing's Wilson Garden — right area → **building**, lat/lng read from the place link (`!3d…!4d…`).
- **Junk name (#2):** `new house` is ungeocodable. Fallback `Nelamangala, Bengaluru` resolved to the place **"Nelamangala Town"** (a *Town*, Bengaluru North District). Centroid read from the resolved `/maps/place/…@…!3d13.097301!4d77.3856398` URL → **locality** (metrics flagged approximate).
- **Address-only (#3):** `Ittina Sarva 1, Bommanahalli, Bengaluru` resolved **directly to the named society** "Ittina Sarva 1 Apartments (ISOWA)" (*Apartment building*, Begur Rd, Hongasandra 560114) → **building**. So a missing building_name does **not** force a locality fallback if the address names the society.

**How to tell building- vs locality-level on Maps:** the place's *type* header is the tell — "Apartment building" / a street address with a house number / a named-society place ⇒ building; a "Town" / neighbourhood / area-only place ⇒ locality. The directions title also reverse-geocodes the origin to a full street address when it's a real building, which is a second confirmation.

**Wrong-region candidates:** the geocode *searches* were clean, but a **destination POI** query bit us: `MedPlus Chennappa Layout Nelamangala, BH Road, Bengaluru` silently resolved to a MedPlus near **Mysore (~150 km away)** and returned an empty route (the signed-in account has Mysore saved places, and the ambiguous "BH Road / Bengaluru" tokens pulled it). Fixed by dropping "Bengaluru" and keeping the town name. **Lesson: always verify the resolved place (title/coords), never trust the raw query string** — the gate must apply to POI destinations too, not just the building.

### 2. Fastest read-path per metric
The **directions deep-link** is the fast path and the core of the recipe:

```
https://www.google.com/maps/dir/<lat,lng>/<destination>/data=!4m2!4m1!3e<MODE>
```

- Origin as raw **`lat,lng`** pins the exact start point (no re-geocoding of the building).
- The **mode bar** at the top shows **every mode's total TIME at once** on a single load (car / two-wheeler / transit / walk / cycle) — times are cheap.
- **DISTANCE** only appears in the route-card list for the *currently selected* mode. **Critical quirk:** clicking a mode icon/radio in the panel does **not** refresh the route list in this automated session (the mode-bar highlight changes but the cards stay stale). The reliable way to get a mode's distance is to **reload with the mode token in the URL** (see Q5 for tokens).
- `get_page_text` reads the route cards cleanly (time + km + "via <road>"); a screenshot is only needed to read the mode bar.
- Net: **1 page-load per mode** needed. walk+car = 2 loads; car+transit = 2 loads; +1 load per "nearest POI" category search. Wrapping `navigate → wait 3s → get_page_text` (and even two modes back-to-back) in one `browser_batch` call is what makes this fast.

### 3. Metro line constraint
**Yes, reliably.** A station's place page shows a **"Metro services: <X> Line"** field. Verified directly: **Lalbagh → Green Line**, **Hongasandra → Yellow Line**; **Madavara → Green Line** (confirmed via its page content/reviews: "northern terminal… North-South corridor of the Green Line"). Searching a unique station name jumps straight to its place page. The `metro station` category search is also distance-ordered (nearest first) and the cluster of names reveals the corridor. All three buildings' nearest stations were on operational lines, and **Yellow Line is confirmed operational**.

### 4. Instamart
**Dark-store pins DO exist** in the city (listed as "Grocery delivery service") — contrary to the brief's expectation:
- #1 Wilson Garden → **Instamart KR Road** (2-wheeler 4.4 km / 12 min).
- #3 Bommanahalli → **Instamart Jp Nagar** (2-wheeler 6.3 km / 18 min).
- #2 Nelamangala → **none in town**; search returned only city stores 30 km+ away → left blank.

So the column **survives for city localities but is genuinely blank for outer towns**. Caveat: even where present, the nearest dark store is several km away (it's a "which store would serve you," not a walkable amenity), and Maps cannot show in-app *serviceability* — pin presence is the only readable proxy.

### 5. Two-wheeler & transit modes
- **Two-wheeler (`!3e9`)** returns values for Bengaluru — verified the motorcycle icon highlighted and a distinct time from car (e.g. 12 vs 13 min). Good.
- **Transit (`!3e3`)** returns values, with quirks:
  - Google returns the fastest **public-transit** option, which for inner-city short hops is usually a **BUS, not metro** (e.g. #1 → Indiranagar fastest transit was 38 min by bus; the Purple-Line metro option was 40 min). For long hauls metro wins (#2 Nelamangala → Indiranagar fastest used Green+Purple metro). **So the captured `mm` value is "transit-as-shown," mixing bus and metro — it is NOT pure walk+metro.** Flag for the user.
  - The transit panel shows **no trip distance** → `mm_km` is unobtainable (blank everywhere).
  - Transit is **schedule-dependent** (showed dated 8 AM departures) — time-of-day sensitive.

### 6. ★ Minutes per building (the feasibility number)
**Steady-state, measured:** **~6.3 min** for building #3 (02:40:24→02:46:42) and **~6.5 min** for #2 (incl. the Mysore wrong-resolution retry) — each for the **full metric set**: geocode + nearest-metro + 3 downtown anchors (car+transit) + 5 daily POIs (walk+car) + Instamart (2-wheeler) ≈ **13 destinations / ~24 directions loads + ~6 category searches**. Building #1 took ~15+ min but that was **method discovery** (fighting the flaky click UI, screenshots) and is not representative.

**Where the time goes:** almost entirely the ~24 directions loads × (navigate + ~3 s settle + read). The geocode+gate is fast (~30–60 s). The `browser_batch` wrapper is what keeps it to ~6 min.

**Verdict — is "all unique buildings" realistic?** At ~6–7 min/building it is feasible *if the unique-building count is modest*. The 728 listings should be **deduped to unique buildings first** (many listings share a society); a plausible few-hundred unique buildings ⇒ on the order of **10–30 hours** of careful, throttle-respecting capture. Recommendations: (a) dedup to unique buildings before any run; (b) consider trimming the lowest-value-per-cost metrics (transit, instamart) — `mm_km` is unobtainable anyway; (c) cache the 3 fixed anchor coords (below); (d) consider scoping to a shortlist of candidate buildings rather than the full set.

### 7. Format / unit quirks
- **Distance units:** ≥1 km shows as `2.3 km`; <1 km shows in **metres** (`550 m`) → converted to km (0.55). Processing must handle both.
- **Time:** `33 min`, or `1 hr 8 min` (→68), or `2 hr 24 min` (→144) → stored as integer minutes.
- **No traffic "ranges"** at this hour — cards gave single live ETAs ("7 min … despite usual traffic"). Multiple route alternatives differ; **I recorded the top / "Best route" (first-listed)**. Note Google's "Best route" is occasionally **not the shortest distance** (e.g. #2 More Supermarket car: Best 8.3 km vs a 5.8 km alt at the same time; #3 grocery car: 1.6 km Best vs an 0.8 km alt). A selection rule should be agreed.
- **Car distance can exceed walk distance** for short hops (one-ways / highway detours) — e.g. #2 hospital walk 1.0 km vs car 3.7 km; #3 grocery walk 0.35 km vs car 1.6 km. Not an error.
- **Time-of-day:** all readings ~08:00 IST (morning rush; sandbox is UTC, IST = UTC+5:30). Driving ETAs are rush-inflated — capture time should be standardized.
- **No CAPTCHA / rate-limit / login wall** across ~70+ Maps loads (browsing signed-in as account "Kanva") with ~3 s spacing between loads. The one `get_page_text` "no content" was the Mysore empty-route, not throttling.
- **Clicking a results-list card to open a place was flaky** (panel often didn't open). Navigating directly to `/maps/place/…` and `/maps/dir/…` URLs is far more reliable than clicking.

## Recommended recipe / schema changes
_(For Claude Code, into `docs/sites/google-maps.md` + geo schema)_

- **Capture method:** directions deep-link `…/maps/dir/<lat,lng>/<dest>/data=!4m2!4m1!3e<MODE>` with mode token **`0`=drive, `2`=walk, `3`=transit, `9`=two-wheeler** (`1`=bicycle); read via `get_page_text`. **Set mode via the URL** — clicking modes in-panel does not refresh the route list. Use `browser_batch` to combine navigate+wait+read (and multiple modes) per call.
- **Geocode + gate:** search `building_name + locality + Bengaluru`; if junk/blank, search the address; last resort the locality. Read coords from the resolved `/maps/place/…!3d<lat>!4d<lng>` URL. Gate by place *type*. **Verify every resolved place (title/coords), including POI destinations** — they can silently resolve to the wrong city (Mysore incident). For POI destinations, prefer a uniquely-named result + town name and **avoid the trailing "Bengaluru"**, or route by the POI's own coords.
- **`*_mm_km`:** Maps transit shows no trip distance → these are **always blank**. Drop the columns or accept permanent blanks.
- **Clarify `mm`:** it's transit-as-shown (bus *or* metro, whichever is fastest), **not** pure walk+metro. Rename, or add an `*_mm_mode` note. Pure-metro time is not directly available from Maps.
- **Route-selection rule:** decide between "top/Best route (what Maps recommends)" and "shortest-distance route" — they diverge in a minority of cases. I used **top/first-listed** for car & the **shortest duration shown** for transit; this should be standardized.
- **POI nearest-selection:** category search returns relevance-ranked (≈ nearest) results — I took the first sensible one. **`supermarket` and `grocery` searches overlap heavily** (same ranked list); the two categories aren't cleanly separable from Maps. Decide whether `supermarket`=big-format (DMart/Reliance/More) and `grocery`=kirana, and search accordingly, or merge them.
- **Standardize capture time-of-day** (ETAs are traffic/schedule dependent). All current data ≈ 08:00 IST rush.
- **Instamart:** keep, but expect blanks for outer localities and multi-km distances even where present.
- **Cache fixed anchor coords** (reusable for every building):
  - Indiranagar Metro → resolves to CMH Rd locality point ≈ **12.9782619, 77.6385257**.
  - M.G. Road Metro → **12.9756295, 77.6066227**.
  - "The Forum Mall, Koramangala" now resolves as **"Nexus Mall Koramangala"** (rebranded) ≈ **12.934692, 77.6111212**.
