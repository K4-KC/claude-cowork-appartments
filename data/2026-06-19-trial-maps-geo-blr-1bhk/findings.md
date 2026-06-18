# Findings — Google Maps geo-enrichment trial `2026-06-19-trial-maps-geo-blr-1bhk`

First-contact trial of **Google Maps** as a geo source. Goal: the **method + feasibility**, not the data. Filled by Cowork (Claude in Chrome) while browsing; Claude Code distills the durable parts into `docs/sites/google-maps.md` and locks the geo schema.

- **Filled by:** _Cowork_ · **Date:** _____ · **Status:** _____ (3 buildings attempted / partial / blocked)

> Capture the 3 buildings into `geo.csv` (already seeded), then answer the questions below. Leave anything Maps won't give **blank** in the CSV and explain it here.

## Outcome

- Buildings captured: _ / 3. Gate tier each landed in — #1 Srinivasa Nilaya: ___ · #2 "new house"/Nelamangala: ___ · #3 Ittina Sarva/Bommanahalli: ___
- Anything you couldn't get for a captured building, and why:

## The 7 trial questions

1. **Geocode reliability + the gate.** Did each path resolve (named building / junk-name→locality / address-only)? How do you *tell* a building-level pin from a locality-level one on Maps? Any wrong-region candidates the gate had to reject?

2. **Fastest read-path per metric.** Best click-path to get distance + time for a mode. Does the Directions panel show all modes (walk/drive/transit/two-wheeler) at once, or one at a time? Can you read distance without opening full directions (e.g. from the Places card)?

3. **Metro line constraint.** Can you reliably see which line (Green/Purple/Yellow) a station is on, so the nearest is kept to the operational lines? Any station ambiguity?

4. **Instamart.** Do Swiggy Instamart dark-store pins exist near any of the 3 buildings? If not, is there *any* readable proxy (serviceability in-app, a listed store)? — this decides whether the column survives.

5. **Two-wheeler & transit modes.** Do both return values for Bengaluru routes? Any oddities (transit missing off-peak, two-wheeler falling back to car)?

6. **★ Minutes per building (the feasibility number).** Roughly how long did one full building take end-to-end? Where did the time go (geocoding vs the ~10 directions lookups)? Is "all unique buildings" realistic, or should we cut the metric set / scope to a shortlist?

7. **Format/unit quirks.** Units ("1.2 km" vs "950 m"), time ranges in traffic ("18–32 min" — which figure did you record?), any rate-limit / CAPTCHA / login prompt on rapid Directions queries.

## Recommended recipe / schema changes
_(For Claude Code: columns that didn't work, metrics too slow to be worth it, anchors that were wrong, gate rules to tighten, etc.)_
