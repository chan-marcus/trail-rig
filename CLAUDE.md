# Trail Rig

Off-road overlanding browser game for the Venture the Terrain brand. A truck "virtual pet" sits on top of an idle expedition game: you outfit a rig, send it on a trail that runs in real time while you are away, then read a trail report and triage the damage.

## Project layout

- `index.html` is the entire game: markup, CSS and JavaScript in one file, no build step, no dependencies beyond Google Fonts (Inter, plus IBM Plex Mono for numbers).
- Hosted on Cloudflare Pages (project `trail-rig`, served at https://trail-rig.pages.dev), deploying from the `main` branch of github.com/chan-marcus/trail-rig.

## How the game works

- Expedition outcomes come from a pure, seeded function: `resolve(rig, ups, start, route, seed)` returns a trail report. Keep it pure and deterministic.
- Balance testing: open the page and run `simulate(3000)` in the browser console to print finish rates and salvage for every rig, terrain and difficulty.
- 3 rigs (Scout, Hauler, Brute), 8 upgrades with 4 slots, 3 terrains (desert, mud, rock), 15 events (5 per terrain), and stranding. Salvage is the only currency.
- Rig level and XP are derived from career stats (`xpOf`, `levelOf`). Level is a readout, not a power gate.
- Save data lives in the browser's localStorage under the key `trail-rig-v2`. Changing the save shape needs a migration or a new key. Adding a field with a default in `fresh()` is safe (that is how the journal's `log` was added).

## Design rules to keep

- Taps, revving, washing, radio check-ins and scrap grabs never change trail outcomes. Outcomes are decided when the rig departs.
- "Grab scrap" buttons during a trail report are capped at 15% of a run's salvage and expire when the report finishes.
- The trail report has the juice (`juice()`): numbers fly off each line, salvage coins fly into the report's counter, finds and 3+ chains burst sparks, and damage gives a small shake. Failures get smaller effects than wins. Skip and the OS reduced-motion setting turn all of it off.
- The main screen stays calm: no confetti, cut-ins, chase lights or tickers. A changed number glows amber briefly (`setVal`), and anything that blinks stays under 3 flashes per second.
- Every amount shown, flying or not, must be a real value that changed. The report counter must land on the run's true salvage plus scrap.

## Look and feel

- Simple, halfway between a modern app and a text game: flat dark rounded cards and real buttons, with the content told in short plain sentences. Inter for text, IBM Plex Mono only for numbers, mile markers and check rolls.
- Palette: charcoal/brown neutrals; amber for the main button, salvage and wins; rust for damage and low gauges; green for passed checks and good fits; blue for XP; colored gauge bars (fuel gold, engine red, suspension blue, tires olive, stoke khaki).
- Camp card: the pixel truck (calm, no particles beyond mud and installs), then the status as one short fact per line, the one obvious next move, pet actions (rev, wash), and the journal of the last 5 events.
- The trail report reads out line by line: one sentence per line (balanced if it wraps), a mono check line, then small effect tags. Hyphenated words and trail names never split across lines (`.pn`).
- Journal entries are one short sentence that fits on one line at 375px, with any salvage change in its own column. Keep trail names out of journal entries.
- Desktop layout is two columns (Camp and Rig on the left, the trail board on the right), stacking to one column under 900px. From 600px to 899px the truck sits beside the camp status. Check 375px, 768px and 1280px for wraps and horizontal overflow after layout or copy changes.
- Do not reproduce real vehicle designs (the pickup sprite is an original design) or real brands.
- Never use em dashes in UI copy.

## Deploy notes

- Pushing to `main` should auto-deploy. As of the "Add favicon and brighter mileage pop-ups" commit, Cloudflare did not pick up the push automatically (the GitHub app has access to all repositories, so the webhook is the suspect). If a push does not appear on the live site within a few minutes, trigger a deploy from the Cloudflare dashboard or add a deploy hook.
