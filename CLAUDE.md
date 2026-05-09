# Morning Inspiration — engineering notes

This file is for Claude (or any agent) helping Thiago evolve the project's code and structure. Editorial direction — what to feature, voice rules, mood, format rotation, exclusions — lives in `EDITORIAL.md`. Leave that file to the editor; only touch it when explicitly asked to.

## What this is

A static personal homepage that displays one curated piece of work per day — a photograph, a film still, a passage, a song. A scheduled Claude Code task runs at 5am and produces a new entry. The site is plain HTML + CSS, served as-is from GitHub Pages. **No JavaScript, no build step, no framework.**

## Repo layout

- `index.html` — today's entry
- `archive/YYYY-MM-DD.html` — past entries, one file per day
- `archive.html` — chronological index of every past entry, regenerated each run
- `history.json` — ledger of every entry (date, mood, format, creator, title, year, source). Drives the cooldowns and the archive page.
- `template.html` — skeleton the generator fills in to produce `index.html`
- `styles.css` — shared stylesheet, used by every page
- `EDITORIAL.md` — editorial direction. The user edits this freely; the daily generator reads it on every run.
- `GENERATOR_PROMPT.md` — the prompt the scheduled agent executes at 5am. Read this if you're changing the daily flow.
- `README.md` — human-facing intro
- `CLAUDE.md` — this file

## Daily flow (high level)

The scheduled agent reads `EDITORIAL.md` + `history.json`, picks a mood (cooldown-aware), then a format (mood-biased), then a specific work. It moves yesterday's `index.html` into `archive/`, generates a new `index.html` from `template.html`, appends the new entry to `history.json`, regenerates `archive.html`, verifies every external URL resolves, and pushes via the GitHub API. Full step-by-step is in `GENERATOR_PROMPT.md`.

## Invariants — don't break these

- **Idempotency.** If `archive/YYYY-MM-DD.html` already exists for yesterday, the day's run already happened — exit without changes.
- **No JavaScript in pages we write.** Self-contained HTML + CSS. Third-party iframe embeds (Bandcamp, YouTube, Spotify, Internet Archive, etc.) are allowed for music/video — they're HTML elements; the JS lives inside the iframe, not in our page.
- **Mood is internal.** Stored in `history.json` to steer selection and prose, but never rendered on the page.
- **Pre-push link verification is mandatory and automated.** Every external URL in `index.html` is fetched before push; broken links cause the work to be silently swapped, not surfaced for review. See Step 12 of `GENERATOR_PROMPT.md`.
- **`styles.css` is shared.** Don't modify it for routine entries — only when `EDITORIAL.md` explicitly asks for a design change.
- **Archived files use `../styles.css`.** When moving `index.html` → `archive/YYYY-MM-DD.html`, the stylesheet href must be rewritten.
- **Cooldowns live in `EDITORIAL.md`.** Format: 3-day window. Mood: 4-day window. Creator: ~30 days. The generator reads these from `EDITORIAL.md` — don't hardcode them in the prompt or anywhere else.

## Editorial vs. mechanical — keep them separate

This is the load-bearing rule for changes to this project. `GENERATOR_PROMPT.md` should stay **content-agnostic**: it covers mechanics (file paths, JSON shape, media markup, verification, push, idempotency). `EDITORIAL.md` carries everything that expresses *taste* — what to feature, the mood wheel, the format wheel, mood↔format pairings, mood↔register pairings, voice rules, exclusions.

A user cloning this repo to run their own dispatch should only need to edit `EDITORIAL.md` to change what gets featured and how it sounds. They should never need to touch `GENERATOR_PROMPT.md`.

Rule of thumb when adding or moving instructions:

- Names a specific creator, mood, format, register, or "this kind of work"? → `EDITORIAL.md`
- Names a file path, JSON field, HTML markup, step in the daily flow, or operational invariant? → `GENERATOR_PROMPT.md`

When in doubt, the test is: *would this instruction make sense in a totally different person's clone of the repo?* If yes, it's mechanical. If it would feel weird or impose your taste on them, it's editorial.

## Where to make different kinds of changes

- *"I want different kinds of content"* → `EDITORIAL.md`
- *"I want the generator to behave differently"* → `GENERATOR_PROMPT.md`
- *"I want a different layout or design"* → `template.html` and/or `styles.css` (and add a note in `EDITORIAL.md` if it changes the visual identity)
- *"I want a new project-level feature"* → usually touches `GENERATOR_PROMPT.md` plus at least one of the above; discuss before implementing

## Push behavior

In the scheduled run, direct `git push origin main` is blocked by branch protection — the agent uses `mcp__github__push_files` instead. When Thiago is working locally, normal `git push origin main` works (with his approval, since main is the default branch).
