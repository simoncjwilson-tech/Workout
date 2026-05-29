# Lift — Workout / Strength Tracker

A custom workout / strength-training tracker for iOS, built because off-the-shelf
apps didn't fit how Simon plans, adapts, and logs gym sessions. Runs as a
standalone PWA added to the iOS home screen.

**Live:** https://simoncjwilson-tech.github.io/workout/
**Current version:** Lift v1.2 — Cloud Sync

## Stack & architecture
- Single self-contained `index.html` (HTML/CSS/JS, no build step, no framework).
- Data in browser `localStorage`. Keys: `lift_v1` (app state), `lift_sync_v1` (sync config).
- Hosted on GitHub Pages from this repo. (Originally conceived as an iCloud Drive
  file launched via Shortcut, but moved to Pages because Safari storage scoping
  for `file://` URLs was unreliable and a stable URL was needed for persistent
  localStorage.)
- Aesthetic: restrained, high-contrast, readable in low gym light, glove-friendly.
  Black / gold / green palette.

## Confirmed design decisions
- Units: metric, kg only.
- Routine "adapt" means both — tweak today's session only, or save changes back to
  the template (per-change toggle).
- Health sync: Apple Health was the intended target via a Shortcut-mediated
  clipboard handoff; deferred, not a direct binding in v1.2.
- Craft as longer-term backend: workflow-mediated (Shortcut + Claude in the loop),
  not a direct API binding — a static HTML page cannot call the Craft MCP server.

## Screens
- `screen-home` — saved routines with last-completed date, "+ New routine", Quick
  start for ad hoc workouts.
- `screen-editor` — name/subtitle; per exercise: name, cue, target sets, rep range,
  default weight (kg), rest seconds; drag to reorder, swipe to delete.
- `screen-session` — vertical exercise list; current exercise expanded with per-set
  weight/reps rows and checkboxes pre-filled from plan; rest timer; progress
  ring/bar; inline edit; completed exercises collapse and next auto-expands.
- `screen-session-detail`
- `screen-summary` — total time, total volume (Σ weight × reps), per-exercise actual
  vs planned, skipped sets.
- `screen-history` — past sessions.
- `screen-settings` — Export all data, Import data, Configure sync.
- `screen-sync` — Cloud Sync config (see below).

## Cloud Sync (Gist sync)
- Cross-device sync via a GitHub Personal Access Token (scope: `gist` only) plus a
  secret gist named `lift.json`.
- Pushes app state to the gist after every workout; on load, if the gist is newer
  than local it restores from it.
- Setup: generate classic PAT (gist scope, no expiry) → create secret gist with
  filename `lift.json` and content `{}` → in app Settings → Configure sync, paste
  token + gist ID → Push to gist now → verify gist URL shows the state JSON.

## Data model
- `Routine`: `{ id, name, subtitle, exercises: [{ id, name, cue, sets, repsMin, repsMax, defaultWeight, restSec }] }`
- `Session`: `{ id, routineId, routineName, startedAt, finishedAt, sets: [{ exerciseName, plannedReps, plannedWeight, actualReps, actualWeight, completed, restSec }], notes }`
- History: array of past sessions in localStorage.

## Seeded content
A strength-focused chest routine (4–6 rep range): Barbell Bench Press (4×40kg),
Incline Dumbbell Press, Chest Fly, Push-Ups (to failure), Tricep Pushdown.

## Deploy / update process
1. Get the latest `index.html`.
2. GitHub repo `Workout` → Add file → Upload files → drop in `index.html` (replaces
   existing) → Commit changes.
3. Wait 30–60s; check the Actions tab for a green checkmark.
4. On phone: open Safari to the URL with a cache-bust query (e.g. `?v=10`) to force
   a fresh fetch.
5. Force-quit the home-screen Lift app and reopen.
6. If still cached: Settings → Safari → Advanced → Website Data → delete the
   github.io entry; last resort, remove and re-add the home-screen icon (this wipes
   localStorage, so export/sync first).

## Known constraints
- localStorage carries an occasional data-loss risk; removing/re-adding the
  home-screen icon wipes it (mitigated once Cloud Sync is configured).
- No auto-progression logic — next-weight suggestion was discussed but not built;
  entry is manual.
- iOS Safari PWA caching is aggressive; updates often need a cache-bust query string.
- The "Lift v1.0" label bug was corrected to v1.2; the version string is cosmetic only.

## Build provenance
Original build conversation: "Custom iOS workout tracking app proposal" —
https://claude.ai/chat/a6496627-aab8-42a5-8e8f-1ac2d1a8356f

---
*This file mirrors the Craft doc "Workout / Lift — App Instructions". The repo is
the source of truth; keep the two in sync.*
