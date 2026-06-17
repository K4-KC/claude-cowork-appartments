# Trial brief — NoBroker (first-contact trial)

- **Run id:** `2026-06-18-trial-nobroker-blr-1bhk`
- **Site:** NoBroker (nobroker.in)
- **Type:** Trial (first contact). Goal is the **method**, not the data — discover the fastest reliable way to capture matching listings, then record it. See `docs/trial-protocol.md`.
- **Driver:** Cowork (Claude in Chrome). Findings come back here for Claude Code to fold into `docs/sites/nobroker.md`.
- **Reference:** `docs/sites/housing.md` and `docs/sites/99acres.md` — recipes built on the **same filters**; note where NoBroker differs.

## Filters for this trial (same as the prior trials, for comparability)

| Filter | Value |
|---|---|
| City | Bengaluru |
| Listing type | Rent |
| BHK | 1 |
| Budget (₹/month) | 12,000 – 22,000 |
| Furnishing | Fully furnished |
| Tenant preference | Bachelors (male & female); "any/all" acceptable if the site can't split it |
| Localities | None specified — city-wide Bengaluru is fine for the trial |

> Throwaway target — exact matches don't matter. If a filter can't be expressed in NoBroker's UI, note the gap in the recipe and move on; we apply it during processing.

## What to do (follow `docs/trial-protocol.md`)

0. **Access** — can you see listings logged out? NoBroker is known for **aggressive login / app-install prompts** and may gate listings or contact behind login/OTP. Record exactly what's blocked without an account, and whether prompts dismiss.
1. **Deep-link probe** — apply the filters once via the UI, then check whether the URL encodes them. If it does, capture the URL template. *(Biggest efficiency win — Housing had it, 99acres did not.)*
2. **Field-coverage map** — for each captured column in `docs/data-schema.md`, note card / detail / login / absent.
3. **Capture 2–3 listings** into `data/2026-06-18-trial-nobroker-blr-1bhk/nobroker.csv`. The header row is already there — fill columns in that exact order, leave unavailable cells blank, never invent a value.
4. **Efficiency verdict** — fastest reliable path, pagination model, main bottleneck, any throttling/quirks.
5. **Record findings** in `data/2026-06-18-trial-nobroker-blr-1bhk/findings.md` (templated with the 7 questions).

## NoBroker-specific things to confirm

- It's an **owner-listings** platform ("no broker"). Confirm whether `brokerage` is genuinely **0 / absent**, and how `posted_by` reads (owner vs the occasional relationship-manager listing).
- Is the **owner's contact reachable without login**, or is it gated like the other sites? (Record as a gap if gated.)
- Any **owner/broker filter** or "verified" badges worth noting.

## Ground rules

Browse like a careful human. Don't bypass logins, CAPTCHAs, or anti-bot measures; don't hammer pagination. Login-walled fields (often `contact_phone`) are recorded as **gaps**, left blank — not worked around. Record the source URL for every listing.

## What to bring back (the 7-item checklist)

Record your answers in `data/2026-06-18-trial-nobroker-blr-1bhk/findings.md` — the shared handback doc, templated with the 7 questions: logged-out access + walls, deep-link URL template, per-filter mapping, field-coverage map, pagination model, fastest path + bottleneck, and any format/unit quirks. Claude Code then distills these into `docs/sites/nobroker.md`.
