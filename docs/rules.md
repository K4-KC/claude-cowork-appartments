# Operating rules

Rules and etiquette for capturing and processing listings. They apply to both the Cowork (browsing) side and the Claude Code (processing) side.

## Capture (Cowork, in Chrome)

- Respect each site's Terms of Service and `robots`. Browse like a careful human, not an aggressive scraper; don't hammer pagination.
- **Pace all repeated requests, not just pagination — including detail-page opens.** Sweeping many result pages or opening many listing detail pages in fast succession trips throttles and anti-bot checks (a 99acres run hit empty-skeleton rate-limiting after ~15 fast pages and a CAPTCHA after rapid detail-page hits — see `docs/sites/99acres.md`). Space requests; capture detail-only fields in small batches.
- Do not bypass logins, paywalls, CAPTCHAs, or anti-bot measures. **A CAPTCHA or throttle is a stop signal, not an obstacle to route around:** if one fires, stop browsing that site, record it in the run's `findings.md`, deliver the partial capture, and resume later at a gentler pace. Partial-but-honest beats triggering harder blocks.
- Capture only what's visible in normal browsing, and record the source URL for every listing.

## Processing (Claude Code, here)

- This repo never browses the sites. If live data is needed, produce or extend a recipe for Cowork instead.
- Treat captured columns as source of truth: validate and flag, but don't silently rewrite values.
- Keep captured and derived data separate: write computed values to new `calc_`-prefixed columns (see `docs/data-schema.md`), never back into a captured column.
- Don't fabricate or interpolate listing data — empty is better than guessed.
- Keep runs isolated under `data/<run-id>/`; don't overwrite prior runs.
- Sites are kept distinct — no cross-site merge or dedup unless the user asks.

## When unsure

- If filters, schema, or calculations are ambiguous, ask the user rather than assuming. This project is being designed iteratively via trial and testing, so flagging an unknown is more useful than guessing.
