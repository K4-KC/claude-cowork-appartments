# Search configuration (per run)

The set of filters for a run, applied identically on every site. The values below are a **template** — confirm the real values with the user when starting a run. Nothing here is a default to assume.

## Core filters

| Field | Example | Notes |
|---|---|---|
| City | Bengaluru | Single city per run (confirm) |
| Localities | Koramangala, HSR | One or more neighborhoods |
| Listing type | Rent | Rent vs buy — this workflow targets rent (confirm) |
| Budget (₹/month) | 25000–45000 | Min–max monthly rent |
| BHK | 2, 3 | One or more configurations |
| Property type | Apartment | Apartment / independent / villa / PG |
| Furnishing | Semi, Full | Unfurnished / semi / fully |
| Min area (sq.ft) | 900 | State whether carpet or built-up |
| Tenant preference | Family | Family / bachelors / any |
| Posted by | Owner | Owner / broker / any |
| Available from | ASAP | Move-in window |

## Notes

- Each site labels these filters differently; mapping a filter to a site's UI controls is part of that site's recipe (`docs/sites/<site>.md`).
- If the user gives a filter a site can't express in its UI, record the gap in the site recipe and apply that filter during the validate/calculate step here instead.
- Record the actual filter values used for a run alongside its data (e.g. a small notes file in `data/<run-id>/`) so a run is reproducible.
