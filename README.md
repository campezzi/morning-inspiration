# Morning Inspiration

A static personal homepage that displays one piece of curated work per day. A new entry appears each morning at 5am — a photograph, a painting, a film still, a passage, a piece of music — with a short note and a link backward to yesterday.

The whole thing is plain HTML and CSS. No JavaScript, no build step, no framework.

## How it works

A scheduled Claude Code task runs daily at 5am. Each run:

1. Reads `EDITORIAL.md` for editorial direction
2. Moves the current `index.html` to `archive/YYYY-MM-DD.html` (yesterday's date)
3. Reads `history.json` to know what's been featured recently
4. Picks a format and a work, following the rotation rules
5. Verifies image and source URLs resolve, then writes a new `index.html` from `template.html`
6. Appends the new entry to `history.json` and regenerates `archive.html`
7. Commits and pushes

The archive grows one file per day. Each page links backward to yesterday and to the full archive.

## Steering the project

Edit `EDITORIAL.md`. That's it.

`EDITORIAL.md` is the living editorial config. The generator reads it on every run. You can change:

- **Current interests** — creators and movements you want to see more of
- **Things I'm tired of** — anything you want excluded for a while
- **Specific pieces** — particular works you'd love to see featured
- **Editorial voice** — how the notes should read
- **Format rotation** — which formats to cycle through, and how
- **Mood rotation** — the wheel of moods and cooldown rules
- **Notes to the generator** — free-form instructions, experiments, overrides

Changes take effect on the next run. If `EDITORIAL.md` and the generator's defaults disagree, `EDITORIAL.md` wins.

(`CLAUDE.md` is a separate file with engineering notes for agents helping evolve the project itself — repo layout, invariants, where things live. You don't need to touch it for content changes.)

## Running manually

If you want to trigger a run outside the schedule, use Claude Code in this repo and give it the prompt from `GENERATOR_PROMPT.md`.

## Deploy to GitHub Pages

1. Create a repo named `yourusername.github.io` on GitHub
2. Push this repo to it:
   ```
   git remote add origin git@github.com:yourusername/yourusername.github.io.git
   git push -u origin main
   ```
3. Go to **Settings > Pages** and set source to the `main` branch
4. The site will be live at `https://yourusername.github.io`

## Set up the scheduled task

1. Go to [claude.ai/code/scheduled](https://claude.ai/code/scheduled)
2. Create a new scheduled task
3. Set cadence: **Daily at 5:00 AM** (your local time)
4. Connect the GitHub repo so the task can read and write to it
5. Copy the prompt from `GENERATOR_PROMPT.md` and paste it in
6. Run once manually to verify everything works before letting it run on schedule

## Repo structure

```
index.html          — today's entry
archive.html        — full chronological index of past entries
archive/            — past entries, one file per day (YYYY-MM-DD.html)
history.json        — ledger of every entry (drives rotation + archive page)
styles.css          — shared stylesheet
template.html       — skeleton the generator fills in
EDITORIAL.md        — edit this to steer what gets featured
CLAUDE.md           — engineering notes for agents working on the project
GENERATOR_PROMPT.md — the prompt for the scheduled task
README.md           — this file
```
