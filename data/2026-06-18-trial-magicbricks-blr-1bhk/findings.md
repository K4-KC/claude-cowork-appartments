# Findings — Magicbricks trial (run `2026-06-18-trial-magicbricks-blr-1bhk`)

**Shared handback doc.** Cowork fills this in while running the trial (`docs/trial-protocol.md`); Claude Code then distills the durable learnings into `docs/sites/magicbricks.md` and validates the captured CSV. This file is the agreed Cowork↔Claude Code communication channel. Keep it factual — record what you actually saw; leave a field blank rather than guessing.

- **Filled by:** Cowork · **Date run:** _TBD_ · **Status:** _not started_
- **Reference:** `docs/sites/housing.md`, `docs/sites/99acres.md`, `docs/sites/nobroker.md` (same filters, already trialled) — call out where Magicbricks differs.

Answer the 7-item checklist from `docs/trial-protocol.md`:

## 1. Logged-out access & walls
_Can you see listings without logging in? Magicbricks is known for aggressive login modals — record each, when it fires, whether it dismisses without an account, and whether it gates listings or contact._

## 2. Deep-linkable search? (URL template)
_After applying the filters via the UI, does the URL encode them? Does reloading it in a fresh tab reproduce the results? Paste the exact URL, then the template with each filter's parameter._

```
URL seen:
URL template:
```

## 3. Filter mapping
_How is each run filter expressed on the site? Which can't be expressed (so we apply them here in processing)?_

| Filter | How set on Magicbricks | Notes / gaps |
|---|---|---|
| City (Bengaluru) | | |
| Listing type (Rent) | | |
| BHK (1) | | |
| Budget (₹12k–22k) | | |
| Furnishing (Fully) | | |
| Tenant (Bachelors) | | |

## 4. Field-coverage map
_For each captured column in `docs/data-schema.md`, mark where it lives: `card` (on the results card) / `detail` (detail page only) / `login` (login-walled — a gap) / `absent`. Pay special attention to the cost set (deposit/maintenance/brokerage/move-in)._

| Column | card / detail / login / absent | Notes |
|---|---|---|
| title, locality, city, rent | | |
| bhk, area, area_basis, furnishing | | |
| deposit, maintenance, brokerage, move_in_charges | | |
| amenities, floor, parking, property_age | | |
| available_from, tenant_preference, posted_by | | |
| contact_name, contact_phone | | |
| building_name, address, bathrooms | | |

_(Add/split rows as needed — every captured column should be accounted for.)_

## 5. Pagination model
_Numbered pages / load-more button / infinite scroll? Roughly how many listings per page?_

## 6. Fastest reliable capture path + bottleneck
_One short paragraph: the quickest normal-browsing route to a full run on this site, and the main bottleneck._

## 7. Format / unit quirks
_Anything processing here must normalize: "₹15k", lakh/crore notation, carpet-vs-built-up area, "Immediately" for `available_from`, sq.ft vs sq.m, posted_by owner/broker labels, etc._

## Sample captured
_How many listings written to `magicbricks.csv`, and any row that needed a `notes` caveat (and why)._
