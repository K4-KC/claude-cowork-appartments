# Operating rules

Rules and etiquette for capturing and processing listings. They apply to both the Cowork (browsing) side and the Claude Code (processing) side.

## Capture (Cowork, in Chrome)

- Respect each site's Terms of Service and `robots`. Browse like a careful human, not an aggressive scraper; don't hammer pagination.
- Do not bypass logins, paywalls, CAPTCHAs, or anti-bot measures.
- Capture only what's visible in normal browsing, and record the source URL for every listing.

## Processing (Claude Code, here)

- This repo never browses the sites. If live data is needed, produce or extend a recipe for Cowork instead.
- Treat captured CSVs as source of truth: validate and flag, but don't silently rewrite values.
- Don't fabricate or interpolate listing data — empty is better than guessed.
- Keep runs isolated under `data/<run-id>/`; don't overwrite prior runs.
- Sites are kept distinct — no cross-site merge or dedup unless the user asks.

## When unsure

- If filters, schema, or calculations are ambiguous, ask the user rather than assuming. This project is being designed iteratively via trial and testing, so flagging an unknown is more useful than guessing.
