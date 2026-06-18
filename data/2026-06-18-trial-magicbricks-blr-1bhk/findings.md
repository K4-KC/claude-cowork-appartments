# Findings — Magicbricks trial (run `2026-06-18-trial-magicbricks-blr-1bhk`)

**Shared handback doc.** Cowork fills this in while running the trial (`docs/trial-protocol.md`); Claude Code then distills the durable learnings into `docs/sites/magicbricks.md` and validates the captured CSV. This file is the agreed Cowork↔Claude Code communication channel. Keep it factual — record what you actually saw; leave a field blank rather than guessing.

- **Filled by:** Cowork · **Date run:** 2026-06-18 · **Status:** ⛔ **BLOCKED → DROPPED from the trial round** (user decision, 2026-06-18)
- **Reference:** `docs/sites/housing.md`, `docs/sites/99acres.md`, `docs/sites/nobroker.md` (same filters, already trialled) — call out where Magicbricks differs.

## ⛔ Blocker — site is not reachable from Cowork (record this; do not treat the trial as done)

The trial could **not** be performed. Every attempt to open `magicbricks.com` through the Cowork (Claude-in-Chrome) browser was refused by a **browser-level safety restriction**, not by the site:

- Tool error returned on navigation: **"This site is not allowed due to safety restrictions."**
- URLs attempted (all refused, identical error):
  - `https://www.magicbricks.com`
  - `https://magicbricks.com/`
  - `https://www.magicbricks.com/property-for-rent-in-bangalore-pppfr`
  - `https://www.magicbricks.com/flats-for-rent-in-bangalore`
- This is an **allowlist/policy block on the `magicbricks.com` domain**, not a network error, login wall, CAPTCHA, or anti-bot measure from the site. It fired before any page content loaded.
- Browser state ruled out as the cause: one Chrome extension instance is connected (`Browser 1`, local, macOS) — the **same** browser the prior three trials (housing.com / 99acres.com / nobroker.in) ran through successfully today. So the block is specific to this domain, not a broken browser.
- Per `docs/rules.md` and Cowork's safety rules, the block was **not** worked around (no curl/python fetch, no computer-use driving of the browser to the site, no archive/cache mirror). Those would defeat the restriction.

### Can the block be allowlisted? (researched 2026-06-18, Anthropic support docs)

- The message matches **Claude in Chrome's _default site blocklist_** — a **server-side** list Anthropic maintains, not tied to this browser or account. Documented blocked categories: **banking / financial-services sites, investment & trading platforms, adult content, crypto exchanges, known piracy sites** ([support: "Using Claude in Chrome safely" → Blocked sites](https://support.claude.com/en/articles/12902428-using-claude-in-chrome-safely)).
- Magicbricks is a **property-rental portal** and fits none of those cleanly. **[Inference]** likely an over-broad / false-positive block — possibly caught by the "financial services" net because the site markets home loans. Not confirmed.
- **No documented self-serve way to un-block a default-blocked site:**
  - **Personal (Pro/Max):** the extension's per-site permissions ("approved sites" / "always allow on this site") govern only sites Claude may _act on_ — they do **not** override the default safety blocklist. No user allowlist for blocked domains exists. **[Unverified]** a community feature request for this is open but unimplemented.
  - **Team/Enterprise:** an Owner can set allowlists/blocklists (Organization settings → Claude in Chrome), but the docs frame these as **additional** restrictions on top of the defaults — they do not state an allowlist can override a default-blocked site. **[Inference/Unverified]** admin allowlisting probably won't reach Magicbricks.
- **Sanctioned route for a wrongly-blocked legitimate site:** report it to Anthropic (in-chat thumbs-down feedback; the safety page also invites reporting category errors/omissions to their support/security contact) and ask for reclassification. Not instant and not user-controlled — **expected behaviour, not guaranteed.**

### Decision

**Magicbricks is DROPPED from this trial round** (user decision, 2026-06-18). Not automatable under the current block; no self-serve fix available. The trial round closes with the **3 completed sites** (Housing, 99acres, NoBroker). Revisit Magicbricks only if the block is later lifted (e.g. reclassification after a report), at which point a normal automated trial can run.

**Consequence:** none of the 7 checklist items below could be observed. They are left blank deliberately — **not** guessed. `magicbricks.csv` holds only its header row (0 listings captured).

---

Answer the 7-item checklist from `docs/trial-protocol.md`:
_(All blank — site could not be reached this session. See blocker above. Do not infer Magicbricks behaviour from the other three sites.)_

## 1. Logged-out access & walls
Not observed — site blocked before any page loaded (see blocker above).

## 2. Deep-linkable search? (URL template)
Not observed — site blocked.

```
URL seen:
URL template:
```

## 3. Filter mapping
Not observed — site blocked.

| Filter | How set on Magicbricks | Notes / gaps |
|---|---|---|
| City (Bengaluru) | | not observed |
| Listing type (Rent) | | not observed |
| BHK (1) | | not observed |
| Budget (₹12k–22k) | | not observed |
| Furnishing (Fully) | | not observed |
| Tenant (Bachelors) | | not observed |

## 4. Field-coverage map
Not observed — site blocked.

| Column | card / detail / login / absent | Notes |
|---|---|---|
| title, locality, city, rent | | not observed |
| bhk, area, area_basis, furnishing | | not observed |
| deposit, maintenance, brokerage, move_in_charges | | not observed |
| amenities, floor, parking, property_age | | not observed |
| available_from, tenant_preference, posted_by | | not observed |
| contact_name, contact_phone | | not observed |
| building_name, address, bathrooms | | not observed |

## 5. Pagination model
Not observed — site blocked.

## 6. Fastest reliable capture path + bottleneck
Not observed — site blocked. The current bottleneck is upstream of browsing: the `magicbricks.com` domain is refused by the Cowork browser's safety restrictions, so no capture path could be tested.

## 7. Format / unit quirks
Not observed — site blocked.

## Sample captured
0 listings written to `magicbricks.csv` (header row only). No rows fabricated — the site could not be reached.
