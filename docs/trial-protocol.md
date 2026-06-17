# Trial protocol: learning how to capture a site

A reusable method for the **first contact** with a site, before its recipe exists. The goal of a trial is **the method, not the data**: discover the fastest reliable way to pull listings that match the filters, then write what you learned into `docs/sites/<site>.md` so the next run is mechanical.

Run one site at a time. A trial is "done" when its site recipe is filled in well enough that a normal run needs no more discovery.

## Who does what

- **Cowork (Chrome)** runs the phases below — it opens the site, applies filters, browses, and captures a few listings into the run CSV.
- **Claude Code (here)** authored this protocol and the run brief, and afterwards folds Cowork's findings into the site recipe and validates the captured CSV against `docs/data-schema.md`.

## Ground rules (from `docs/rules.md`)

- Browse like a careful human. Don't bypass logins, paywalls, CAPTCHAs, or anti-bot measures, and don't hammer pagination.
- The "fast path" we're hunting for must be a **normal-browsing** path (a deep-linkable URL, richer cards, fewer clicks) — **not** scripted scraping of hidden/private APIs or anything that defeats a wall.
- Capture only what's visible in normal browsing. Record the source URL for every listing. If a field is behind a login, record that as a *gap*, don't work around it.
- Don't fabricate. A field the site doesn't show is left blank (see `docs/data-schema.md` conventions).

## Phases

### Phase 0 — Access & entry point
- Can you browse and see listings **logged out**? Note any login/app-install modal, when it fires, and whether it can be dismissed without an account.
- Find the search entry point for the run's listing type (rent). Note the starting URL.

### Phase 1 — Deep-link probe (the biggest efficiency lever)
Apply the run's filters **once** through the UI, then inspect the resulting URL.
- Are the filters encoded in the URL (query params and/or path segments)? Reload the URL in a fresh tab — does it reproduce the same filtered results?
- If yes: write down the **URL template** with each filter's parameter. This is the win — future runs skip the UI entirely and just open the URL.
- If no (filters held in session/cookies only): note that, and record the exact click-path instead.

### Phase 2 — Field-coverage map
For each **captured column** in `docs/data-schema.md`, classify where it lives:
- `card` — visible on the results/list card (cheap; no detail page needed)
- `detail` — only on the listing's detail page (costs a click per listing)
- `login` — only after logging in / revealing contact (a **gap** — leave blank, don't bypass)
- `absent` — this site never shows it

This map decides the capture strategy: if most fields are on the card, capture from the list and only open detail pages for the few that need it.

### Phase 3 — Capture 2–3 listings end-to-end
Pick the first 2–3 matching listings and capture every available field into `data/<run-id>/<site>.csv` (the header row is already there — fill columns in that exact order; leave unavailable cells blank). This validates that the schema actually maps to the real page and surfaces unit/format quirks (e.g. "₹15k", "1.2 Cr", carpet-vs-builtup area, "Immediately" for `available_from`).

### Phase 4 — Efficiency verdict
Write down, in one short paragraph, the **fastest reliable path** to a full run on this site, plus:
- Pagination model: numbered pages / "load more" button / infinite scroll — and roughly how many listings per page.
- The main bottleneck (e.g. "phone is login-walled", "must open each detail page for area").
- Any throttling, interstitials, or layout variants seen.

### Phase 5 — Record learnings
Cowork writes its raw findings into the run's **shared handback doc** `data/<run-id>/findings.md` (a template pre-filled with the checklist below) — the agreed Cowork↔Claude Code channel. Claude Code then distills the durable learnings into `docs/sites/<site>.md`: the URL template + filter mapping (Phase 1), the field-coverage map (Phase 2), pagination + open-a-listing notes (Phase 3–4), and gotchas (login walls, popups, quirks). The captured CSV and `findings.md` stay in the run folder as evidence.

## Findings checklist — every trial must answer these

1. Does logged-out browsing work, and what wall (if any) interrupts it?
2. Is the filtered search **deep-linkable**? If so, the exact URL template.
3. How is **each run filter** expressed on this site (and which can't be, so we apply them here in processing)?
4. Which captured columns are on the **card** vs only on the **detail page** vs **login-walled** vs **absent**?
5. What's the **pagination** model, and how many results per page?
6. What's the **fastest reliable capture path**, and what's the bottleneck?
7. Any **format/unit quirks** that processing here will need to normalize?

## Definition of done

The site's recipe (`docs/sites/<site>.md`) answers all seven checklist items, a 2–3 row sample CSV validates the schema mapping, and the gotchas section warns the next run about whatever bit us.
