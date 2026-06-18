# Run brief — NoBroker (nobroker.in, normal run)

- **Run id:** `2026-06-18-nobroker-blr-1bhk-furnished-bachelors`
- **Site:** NoBroker (nobroker.in)
- **Type:** Normal run (recipe exists: `docs/sites/nobroker.md`).
- **Driver:** Cowork (Claude in Chrome). Findings → `findings.md`.

## Filters

| Filter | Value |
|---|---|
| City | Bengaluru |
| Listing type | Rent |
| BHK | 1 BHK only (`type=BHK1`; **1 RK excluded** — NoBroker lists it as a separate button) |
| Budget (₹/month) | 12,000 – 22,000 (set via URL `rent=12000,22000`) |
| Furnishing | Fully Furnished (`furnishing=FULLY_FURNISHED`) |
| Tenant preference | **Bachelors men AND women** — keep listings open to both men and women, OR "any/all/anyone". **Exclude** listings restricted to only men or only women. |

## Localities (NoBroker is per-locality — no city-wide run)

NoBroker's results list fails (silent 501, perpetual skeletons) without a per-locality geo-token, so there is no city-wide search (see `docs/sites/nobroker.md`). User chose the **Top 8 bachelor hubs**, each run separately then combined into one `nobroker.csv`:

1. BTM Layout
2. Koramangala
3. HSR Layout
4. Marathahalli
5. Whitefield
6. Bellandur
7. Electronic City
8. Indiranagar

## User decisions (2026-06-18)

- **Localities:** Top 8 bachelor hubs above (user pick).
- **Depth:** **Full schema** — open each listing's detail page for the ~5 detail-only fields (bathrooms, floor, property_age, parking, building_name) in addition to all card fields.
- **Volume:** capture **all matching listings** per locality (walk the full infinite-scroll set).
- **Gender rule:** see below.

## Tenant rule (how the gender constraint is applied)

NoBroker's lease-type filter is **split into Bachelor Male + Bachelor Female** (no single "Bachelors") — see `docs/sites/nobroker.md`. So:
- Apply both `leaseType=BACHELOR_MALE,BACHELOR_FEMALE` in the URL (OR semantics — returns listings open to male OR female bachelors).
- On capture, record `tenant_preference` exactly as shown (the recipe notes it usually prints "Anyone"/"All" — bachelor-eligible ⊇ Anyone).
- **Exclude** any listing whose stated preference restricts to a single gender (e.g. "Bachelor (Male only)" / "Men only" / "Female only"). Keep listings open to bachelors generally (both genders) or "any/anyone". If a listing shows no single-gender restriction, treat it as open to both → keep.

## Per-locality / radius note

`radius` defaults to 2.0 km and bleeds into neighbouring localities (a Koramangala search surfaced a BTM Layout listing in the trial). Capture the listing's actual locality as shown; flag radius-bleed rows in `notes`. With 8 overlapping localities, **de-dup by `listing_id` across localities** during processing (record first-seen locality).

## Ground rules

Browse like a careful human; pace detail-page opens (NoBroker showed no CAPTCHA/throttle in the trial, but space requests). Don't bypass logins/OTP. `contact_name` + `contact_phone` are phone+OTP-walled → gaps, left blank. `brokerage` = 0 (platform-wide "Without Brokerage"). `posted_by` = owner is an **[Inference]** (no printed per-listing label). Record source URL per listing. Never invent values.
