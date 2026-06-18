# Recipe: Magicbricks (magicbricks.com)

How Cowork should browse and extract from Magicbricks. **To be filled in via trial and testing** — record what works, what breaks, and the exact UI steps as you learn them.

## Trial status

- **Status:** ⛔ **BLOCKED → DROPPED from the trial round** (user decision, 2026-06-18). The first-contact trial could not be performed — see Blocker below. Raw record: `data/2026-06-18-trial-magicbricks-blr-1bhk/findings.md`.
- The trial round closed with the 3 reachable sites (`docs/sites/housing.md`, `docs/sites/99acres.md`, `docs/sites/nobroker.md`). 
- Recipe sections below are **unfilled** — nothing was observed; do **not** infer Magicbricks' behaviour from the other three sites.

## Blocker — site not reachable from Cowork

`magicbricks.com` is refused by a **browser-level safety restriction** in Cowork (Claude in Chrome), not by the site:

- Navigation error: **"This site is not allowed due to safety restrictions."** It fires before any page loads, on every `magicbricks.com` URL tried (www, bare, and rent SRP paths).
- It is **not** a network error, login wall, CAPTCHA, or anti-bot measure. The same browser ran housing.com / 99acres.com / nobroker.in successfully the same day, so the block is **domain-specific**.
- Matches **Claude in Chrome's default (server-side) site blocklist** (documented categories incl. banking/financial-services, trading, adult, crypto, piracy — per Anthropic support). Magicbricks is a property portal; **[Inference]** likely an over-broad / false-positive match (possibly via its home-loan marketing). Unconfirmed.
- Per `docs/rules.md`, the block was **not** worked around (no curl/python fetch, no archive mirror, no driving the browser around the restriction).
- **No documented self-serve un-block:** per-site extension permissions don't override the default blocklist; Team/Enterprise admin allowlists are framed as *additional* restrictions, not overrides **[Unverified]**. Sanctioned route: report the misclassification to Anthropic (in-chat feedback) and request reclassification — not instant, not user-controlled.

## Revisit condition

Re-run the standard first-contact trial (`docs/trial-protocol.md`) **only if** the domain block is later lifted (e.g. after a reclassification report). Until then the sections below stay unfilled.

## URL / entry point

- _Not observed — site blocked._

## Applying the run's filters

- _Not observed — site blocked._

## Walking results

- _Not observed — site blocked._

## Extraction → CSV

- _Not observed — site blocked._ `magicbricks.csv` holds only its header row (0 listings; nothing fabricated).

## Gotchas

- **The blocker itself is the headline gotcha** — the site is unreachable from Cowork under the current safety blocklist (see Blocker above).

## Findings log

Dated, raw observations from each trial/run — the working notes that get distilled into the sections above. Newest first.

| Date | Observation | Action taken / open question |
|---|---|---|
| 2026-06-18 | Trial could not run — `magicbricks.com` refused by Cowork's browser safety blocklist ("site not allowed due to safety restrictions"); domain-specific, not the site. | **Dropped from the round** (user decision). Revisit only if the block is lifted; optionally report the misclassification to Anthropic for reclassification. |
