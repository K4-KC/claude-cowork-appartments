# Run brief — Housing.com (normal run)

- **Run id:** `2026-06-18-housing-blr-1bhk-furnished-bachelors`
- **Site:** Housing.com (housing.com)
- **Type:** Normal run (recipe exists: `docs/sites/housing.md`).
- **Driver:** Cowork (Claude in Chrome). Findings → `findings.md`.

## Filters

| Filter | Value |
|---|---|
| City | Bengaluru (city-wide, all localities) |
| Listing type | Rent |
| BHK | 1 RK **and** 1 BHK (Housing.com lists them separately — both selected; RK rows flagged in `notes`) |
| Budget (₹/month) | 12,000 – 22,000 |
| Furnishing | Fully Furnished |
| Tenant preference | **Bachelors men AND women** — keep listings open to both men and women, OR "any/all". **Exclude** listings restricted to only men or only women. |

## User decisions (2026-06-18)

- **Volume:** capture **all matching listings, every page** (walk the full infinite-scroll set).
- **Depth:** **full schema** — open each listing's detail page + price-breakup popover; fill all 31 captured columns.
- **1 RK studios:** **include** them but **flag** each in `notes` (user asked for 1bhk; Housing.com lists 1 RK separately, so both filters set).

## Tenant rule (how the gender constraint is applied)

Housing.com's lease-type filter has a single **Bachelor** option (not split male/female) — see `docs/sites/housing.md`. So:
- Apply the **Bachelor** lease-type filter in the UI.
- On capture, record the listing's `tenant_preference` exactly as shown (slash-set, e.g. "Family/Company/Bachelor").
- **Exclude** any listing whose stated preference restricts to a single gender (e.g. "Bachelor (Men)" / "Boys only" / "Female only"). Keep listings that accept bachelors generally (both genders) or "any". If a listing shows no gender restriction, treat it as open to both → keep.

## Ground rules

Browse like a careful human; don't hammer pagination. Don't bypass logins/CAPTCHAs. `contact_phone` is login/lead-gated → gap, left blank. Record source URL per listing. Never invent values.
