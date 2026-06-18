# Run brief — 99acres (normal run)

- **Run id:** `2026-06-18-99acres-blr-1bhk-furnished-bachelors`
- **Site:** 99acres (99acres.com)
- **Type:** Normal run (recipe exists: `docs/sites/99acres.md`).
- **Driver:** Cowork (Claude in Chrome). Findings → `findings.md`.

## Filters

| Filter | Value |
|---|---|
| City | Bengaluru (city-wide, all localities) |
| Listing type | Rent |
| BHK | 1 BHK (99acres bundles 1 RK/1 BHK in one filter) |
| Budget (₹/month) | 12,000 – 22,000 |
| Furnishing | Fully furnished ("Furnished") |
| Tenant preference | **Bachelors men AND women** — keep listings open to both men and women, OR "All". **Exclude** listings restricted to only men or only women. |

## User decisions (2026-06-18)

- **Volume:** capture **all matching listings, every page** (walk the full infinite-scroll set).
- **1 RK studios:** **include** them but **flag** each (99acres' "1 RK/1 BHK" filter bundles studios; user asked for 1 BHK).
- **Tenant rule applied in processing:** the "Available for" filter (Single Men + Single Women) returns the *union* (men- or women-eligible). Keep only rows whose detail-page `tenant_preference` is "All" or "Bachelors (Men/Women)"; drop any men-only or women-only.

## Ground rules

Browse like a careful human; don't hammer pagination. Don't bypass logins/CAPTCHAs. `contact_phone` is login-walled → gap, left blank. Record source URL per listing. Never invent values.
