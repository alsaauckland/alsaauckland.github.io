# Instructions for AI assistants working on this repo

This is the ALSA Auckland website (static GitHub Pages site, served at
alsaauckland.org). **Read `HANDOVER.md` first** — it documents the whole site:
structure, design system, the Google Apps Script form backends, deployment, and
the gotchas. Everything you need to work here is in that file.

## Standing rule: keep HANDOVER.md current

`HANDOVER.md` is the single source of truth for how this website works, written
so a non-expert can maintain it. **Whenever you make a change that alters how the
site is built, structured, or maintained, update `HANDOVER.md` in the same commit.**

Update it when you:
- add, remove, or rename a page or a whole section
- add or change a form, its Apps Script backend, or an `/exec` endpoint
- introduce or change a reusable system (events auto-expiry, gallery data model,
  deadline auto-close, waitlist logic, etc.)
- change a sitewide convention (design tokens, nav/footer, deployment, house style)
- discover a new gotcha worth warning the next person about

Do **not** clutter it with one-off content edits (fixing a typo, swapping a photo,
changing an event date). It documents *how the site works*, not every edit.

Keep it accurate and concise. If something in HANDOVER.md becomes wrong, fix it
rather than letting it drift. Treat "update the handover" as part of finishing any
structural change, the same way you'd update a test.

## Other essentials (full detail in HANDOVER.md)

- **Push to `main` = deploy.** There is no separate deploy step for the website.
- **Apps Script: save ≠ deploy.** Redeploy a *new version* for script changes to go live.
- **Script source of truth:** `~/Documents/alsa-email-scripts/*.gs`. Edit there,
  paste into the Apps Script editor — never copy code out of a chat window (it
  corrupts spaces/quotes). Validate with `node --check`.
- **Root-absolute asset paths** (`/images/...`). Preview via a local server.
- **House style:** NZ/British spelling, no em dashes, no AI filler, confident tone,
  bold-lead bullet format.
- **Deadlines close at 9:00pm and pages must auto-close** (see the deadline
  pattern in HANDOVER.md §8).
