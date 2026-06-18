# Cowork — standing run instructions (normal runs)

The durable, generic playbook Cowork (Claude in the Chrome extension) follows for a **normal run** of any site whose recipe already exists. It is the standing replacement for the per-run kickoff message: for a normal run **no kickoff is generated** — a human points Cowork at this file (or straight at a site recipe) and names the run.

> **Kickoff messages are a design-time tool only.** The copy-pastable `tmp/cowork-kickoff-<run-id>.txt` messages exist to *build* a site's recipe — i.e. first-contact trials (`docs/trial-protocol.md`), where there is no recipe to point at yet. Once `docs/sites/<site>.md` is filled in, runs of that site need no kickoff. `tmp/` is git-ignored and transient.

## What you need to start a run

Three inputs, from the human or from the run's brief:

1. **Site(s)** — which of housing / 99acres / nobroker / magicbricks. (Magicbricks is currently blocked — see its recipe.)
2. **Run id** — the folder all output goes under, `data/<run-id>/` (e.g. `2026-07-01-koramangala-2bhk`).
3. **Filters** — the search parameters. Read them from `data/<run-id>/BRIEF.md` if that file exists; otherwise ask the user. **Confirm them before browsing.** (Template: `docs/search-config.md`.)

## Read order (auto-load)

For each site, before browsing, read in this order:

1. **This file** — `docs/cowork-run.md` (the standing procedure).
2. **The site recipe** — `docs/sites/<site>.md` (URL template, filter mapping, field-coverage map, gotchas). This is the main guide.
3. **`docs/data-schema.md`** — the exact columns to capture and their units/conventions.
4. **`docs/rules.md`** — browsing etiquette and the don't-bypass-walls rules.

That's everything; a site whose recipe is filled in needs no other file.

## Do, per site

1. **Open the site** via the recipe's entry point. Re-use a deep-link URL template where the recipe gives one; otherwise apply the filters in the UI exactly as the recipe's filter-mapping table describes (mind the per-site quirks it flags).
2. **Walk the results** per the recipe's pagination notes, opening detail pages only for the fields the coverage map marks `detail`.
3. **Capture** each matching listing into `data/<run-id>/<site>.csv`, using the **exact column order** in `docs/data-schema.md`. Leave login-walled / absent fields blank — **never invent a value**. Record the source URL for every listing.
4. **Hand back learnings** in `data/<run-id>/findings.md`: anything that changed since the recipe was written (URL drift, a new login modal, a layout change, a filter that moved) so Claude Code can refresh `docs/sites/<site>.md`. If nothing diverged, say so — that confirmation is useful too.

## Ground rules (full text in `docs/rules.md`)

- Browse like a careful human; don't hammer pagination.
- Don't bypass logins, paywalls, CAPTCHAs, or anti-bot / safety blocks. A login-walled field is a **gap** — leave it blank, don't work around it.
- Capture only what's visible in normal browsing. Don't fabricate.

## If the recipe doesn't fit (or doesn't exist)

If a site has no recipe yet, or its recipe no longer matches the live site, **stop and switch to a first-contact trial** (`docs/trial-protocol.md`) rather than guessing — that is the design-time path, and it is the one case that uses a kickoff. Record what diverged in `data/<run-id>/findings.md` so Claude Code can rebuild the recipe.
