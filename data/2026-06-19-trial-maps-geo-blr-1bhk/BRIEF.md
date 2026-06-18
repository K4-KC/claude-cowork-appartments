# Trial brief — Google Maps as a geo-enrichment source (first contact)

- **Run id:** `2026-06-19-trial-maps-geo-blr-1bhk`
- **Source:** Google Maps (maps.google.com) — a **new** capture source, no recipe yet.
- **Type:** First-contact **TRIAL** (`docs/trial-protocol.md`). Goal is the **method, not the data** — find the fastest reliable way to read each geo metric off Maps, validate the geocode gate, and **measure minutes-per-building**. Capturing just the **3 buildings** below is enough.
- **Driver:** Cowork (Claude in Chrome). Findings → `findings.md` in this folder.
- **Why this exists:** the user wants geo-derived data joined onto the captured rental listings (728 across 99acres/housing/nobroker). Those distances aren't in any CSV and Claude Code can't browse — so **Cowork reads them off Google Maps** and writes them into `geo.csv`. This trial decides whether that's feasible at scale before we run all unique buildings.

## What we're capturing: one row per building

Output file: **`data/2026-06-19-trial-maps-geo-blr-1bhk/geo.csv`** (header already written; 3 rows pre-seeded with the source listing each building came from — fill the rest left-to-right, leave anything you can't get **blank**, never invent a number).

A geo row describes a **physical building**, geocoded once and (later, at full scale) reused across every listing in that building. For the trial each row maps to one source listing for traceability.

## The 3 test buildings (span the geocode difficulty range)

| # | geo-path to test | source_site / id | what to geocode | open if needed |
|---|---|---|---|---|
| 1 | **building-level (best case)** | nobroker / `8a9f92d673cd3a7b0173cda4bcc834dd` | **"Srinivasa Nilaya", Wilson Garden** — addr "Lakkasandra Extension, near Akkayamma Devi Temple, Wilson Garden". | https://www.nobroker.in/property/1-bhk-apartment-for-rent-in-wilson-garden-bangalore-for-rs-20000/8a9f92d673cd3a7b0173cda4bcc834dd/detail |
| 2 | **locality-only / junk name** | 99acres / `O91951244` | building_name is **"new house"** (ungeocodable) — fall back to locality **Nelamangala, Bangalore West**. | https://www.99acres.com/studio-apartment-flat-for-rent-in-nelamangala-bangalore-west-600-sqft-spid-O91951244 |
| 3 | **address-fallback (no building_name)** | housing / `6887183` | building_name is **blank** — geocode from address **"Ittina Sarva 1, Bommanahalli, South Bangalore"**. | https://housing.com/rent/6887183-675-sqft-1-bhk-apartment-on-rent-in-bommanahalli-bengaluru-v2 |

## Step 1 — Geocode + the confidence gate (do this first for each building)

Search Maps for the building (try `building_name + locality + Bengaluru`; if that's junk/blank, use the address; last resort, the locality). Then **judge the match and record `geo_confidence`** — this gates everything else:

- **`building`** — Maps lands on the actual named society / street address, clearly in the right locality of Bengaluru. Record `lat`,`lng` of that pin. Proceed to capture all metrics.
- **`locality`** — only the neighborhood resolves (name was junk/missing, or several candidates). Record the **locality-centroid** lat/lng and set confidence `locality`. You may still capture metrics, but they're **approximate** — note that in `geo_notes`.
- **`none`** — you can't confidently place even the locality, or the only match is in the wrong city/region. Leave `lat`,`lng` **blank**, set confidence `none`, and **leave all distance fields blank**. Do not guess a pin.

Read lat/lng from the URL or right-click → "What's here?". Put any caveat (which fallback you used, ambiguity, wrong-region candidates seen) in `geo_notes`.

## Step 2 — Capture each metric off Maps (only if confidence is `building` or `locality`)

For each target below, get **distance (km)** and **time (min)** for the stated travel mode, reading Maps' Directions panel. Record the **name** of the chosen nearest place where the column asks for it.

| Target group | Columns | How to find / mode |
|---|---|---|
| **Nearest metro** (Namma Metro **Green / Purple / Yellow** lines only — the operational lines) | `metro_name`, `metro_line`, `metro_walk_km/min`, `metro_car_km/min` | Search "metro station" near the pin; pick the nearest station **on one of those 3 lines** (Maps shows the line). Directions: **Walking** + **Driving**. |
| **Indiranagar / MG Road / Koramangala** — by car AND by metro | `<area>_car_km/min` (Driving), `<area>_mm_km/min` (**Transit** = walk+metro) | Route to a fixed anchor per area: **Indiranagar Metro Station**, **M.G. Road Metro Station**, **Koramangala (Forum Mall)**. `mm` = Google **Transit** mode total (walk+metro), as discussed — Maps won't route car+metro as one trip, so transit-as-shown is the figure. |
| **Daily-needs POIs** — nearest of each, by foot AND car | per category: `<cat>_name`, `<cat>_walk_km/min`, `<cat>_car_km/min` | Search the term near the pin, take the nearest sensible result. Modes: **Walking** + **Driving**. |
| **Instamart** | `instamart_name`, `instamart_2w_km/min` | Search "Swiggy Instamart" near the pin; nearest pin if one exists. Mode: **Two-wheeler** (Google has a motorcycle mode in India). **Expect this to often fail** — Instamart runs delivery-only dark stores that may not appear as map pins. If none found, leave blank and say so. |

**POI search terms** (use these so categories stay consistent): hospital → "hospital"; pharmacy → "pharmacy" / "medical store"; gym → "gym" / "fitness center"; supermarket → "supermarket" (big format: DMart / Reliance Smart / More); grocery → "grocery store" / "provision store" / "kirana".

## What to record in findings.md (the 7 trial questions, geo-adapted)

1. **Geocode reliability** — did each of the 3 paths (named building / junk-name→locality / address-only) resolve? Which gate tier did each land in? How do you *tell* building-level from locality-level on Maps?
2. **Fastest read-path per metric** — best click-path to get distance+time for a mode (e.g. does the Directions panel show all modes at once? can you read distance without opening full directions?).
3. **Metro line constraint** — can you reliably see which line a station is on, to keep to Green/Purple/Yellow?
4. **Instamart** — do dark-store pins exist at all near these 3 buildings? If not, is there any readable proxy?
5. **Two-wheeler & transit modes** — are both available/return values for Bengaluru?
6. **★ Minutes per building** — roughly how long did one full building take, end to end? This decides whether "all unique buildings" is realistic. Note where the time goes.
7. **Format/quirks** — units ("1.2 km" vs "950 m"), time ranges ("18–32 min" in traffic — which figure to take?), any rate-limit/CAPTCHA on rapid Directions queries, login prompts.

## Ground rules (`docs/rules.md`)

Browse like a careful human; space out queries (rapid automated Directions calls can throttle). Don't bypass logins/CAPTCHAs. Read only what Maps shows in normal use; don't script the Directions/Places API. **Never invent** a coordinate, distance, or time — blank is the correct answer when Maps won't give it. Record what blocked you as a gap.
