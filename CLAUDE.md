# Trail Rig

Off-road overlanding browser game for the Venture the Terrain brand. A truck "virtual pet" sits on top of an idle expedition game: you outfit a rig, send it on a trail that runs in real time while you are away, then read a trail report and triage the damage.

## Project layout

- `index.html` is the entire game: markup, CSS and JavaScript in one file, no build step, no dependencies beyond Google Fonts (Inter, plus IBM Plex Mono for numbers).
- Hosted on Cloudflare Pages (project `trail-rig`, served at https://trail-rig.pages.dev), deploying from the `main` branch of github.com/chan-marcus/trail-rig.

## How the game works

- Expedition outcomes come from a pure, seeded function: `resolve(rig, ups, start, route, seed)` returns a trail report. Keep it pure and deterministic.
- Balance testing: open the page and run `simulate(3000)` in the browser console to print finish rates and bolts for every rig, terrain and difficulty.
- Trail lengths and real-time durations (`LEN`): tutorial 1 min, short 2.5 min, day trip 30 min, long haul 2 h.
- 3 rigs (Scout, Hauler, Brute), 8 upgrades with 4 slots, 3 terrains (desert, mud, rock), 15 events (5 per terrain), and stranding. Bolts are the only currency. Players always see "bolts" (use `bolts(n)` for amounts so 1 reads "1 bolt"); the code and save data still call it `salvage`, so leave those keys alone.
- Map: the 15 named trails (`NAMES`) sit on a fog-of-war map (`mapSVG`, positions in `MAPPT`). The first run on a trail pays a discovery bonus (`DISC`, 20 to 40 bolts by difficulty), and mapping all 5 trails in a region pays `REGION_BONUS` (100) once. Both are paid in `readReport` via `discover()`, outside `resolve`, so outcomes stay pure. Map progress saves under `st.map` (`seen`, `regions`). The map is always on screen in its own card above Camp (`renderMap`). During a trip it draws the route from camp and a glowing truck that moves along it in real time (`moveTruck`, updated every second). `genRoutes` leans 70% toward trails you have not found yet, and the board tags undiscovered trails with their bonus.
- Rig level and XP are derived from career stats (`xpOf`, `levelOf`). Level (shown as LVL) never gates anything, but each level reached pays `LVREWARD(l)` = 25 x l bolts once, in the trail report (tracked by `st.lvPaid`). The header shows a big level badge, the XP bar, and how much XP is left plus the next reward.
- Save data lives in the browser's localStorage under the key `trail-rig-v2`. Changing the save shape needs a migration or a new key. Adding a field with a default in `fresh()` is safe (that is how the journal's `log` was added).

## Design rules to keep

- Taps, revving, washing, radio check-ins and scrap grabs never change trail outcomes. Outcomes are decided when the rig departs.
- "Grab scrap" buttons during a trail report are capped at 15% of a run's bolts and expire when the report finishes.
- The trail report has the juice (`juice()`): numbers fly off each line, bolt coins fly into the report's counter, finds and 3+ chains burst sparks, and damage gives a small shake. Failures get smaller effects than wins. Skip and the OS reduced-motion setting turn all of it off.
- The main screen stays calm: no confetti, cut-ins, chase lights or tickers. A changed number glows amber briefly (`setVal`), and anything that blinks stays under 3 flashes per second.
- Every amount shown, flying or not, must be a real value that changed. The report counter must land on the run's true bolts plus scrap.

- Sounds are synthesized truck noises (Web Audio, no files): `vroom` engine revs, `blip` throttle (pitch rises with the chain), `horn`, `hiss` air brake, `clunk`, `crunch` for damage, `sputter` when stranded, `ratchet` wrench, `glug` fuel, `rattle` loose bolts, `squelch` CB radio and `gravel`. Keep new sounds in that family. The sound toggle mutes all of them.

## Look and feel

- Simple, halfway between a modern app and a text game: flat dark rounded cards and real buttons, with the content told in short plain sentences. Inter for text, IBM Plex Mono only for numbers, mile markers and check rolls.
- Palette: charcoal/brown neutrals; amber for the main button, bolts and wins; rust for damage and low gauges; green for passed checks and good fits; blue for XP; colored gauge bars (fuel gold, engine red, suspension blue, tires olive).
- Camp card: the pixel truck (calm, no particles beyond mud and installs), then the status as one short fact per line, the one obvious next move, pet actions (rev, wash), and the journal of the last 5 events.
- The trail report reads out line by line: one sentence per line (balanced if it wraps), a mono check line, then small effect tags. Hyphenated words and trail names never split across lines (`.pn`).
- Journal entries are one short sentence that fits on one line at 375px, with any bolt change in its own column. Keep trail names out of journal entries.
- The rig sheet is a full-width header card at the top (`.righead`): name, level and XP beside four compact gauges from 900px, gauges on their own row from 600px, and the gear slots in a row below. Phones get the stacked list version.
- One screen, no scrolling on desktop (900px+ wide, 620px+ tall): the page locks to the window height, the truck screen sizes from the window height, the journal shows as many lines as fit, and every tab (Trails, Workshop, Ledger) must fit its card without scrolling at 1024x700, 1366x768, 1280x800 and 1440x900. Use compact tiles (`.tile`, `.grid2`, `.grid3`) for new content. Under 860px tall, a dense mode folds two-line details onto one line. The board's panel only scrolls internally as a fallback on very short windows. Phones and tablets scroll normally.
- Below it, desktop is two columns (the map card and a compact Camp card stacked on the left, the trail board on the right; the map takes whatever height is left), stacking to one column under 900px. From 600px to 899px the truck sits beside the camp status. Check 375px, 768px and 1280px for wraps and horizontal overflow after layout or copy changes.
- Do not reproduce real vehicle designs (the pickup sprite is an original design) or real brands.
- Never use em dashes in UI copy.

## Deploy notes

- Pushing to `main` auto-deploys to https://trail-rig.pages.dev, usually within 30 seconds (working reliably since 2026-10-15). Commit and push finished, verified changes to `main` without asking. Afterwards, check that the live page serves the new version.
