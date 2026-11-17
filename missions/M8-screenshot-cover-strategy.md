# M8 — Screenshot & cover strategy

- **Target criteria**: completeness (1), cover impact (1) — **Priority P0** — Owner: Luna (spec) → director/implementing agent executes the shots
- **Files**: `shots.json`, `screenshots/`, `game/src/main.js` (only if a new shot seed is needed)

## Context

The judge sees ONE contact sheet: the first screenshot is the cover. Currently: 1-title, 2-play (mid-fight LEVEL 2), 3-boss (LEVEL 3 Hollow Crown), 4-victory. They are legible; none is unforgettable.

## Deliverable (spec)

1. **Cover concept** — exactly one title-frame composition: the dying sun dominating, the tiny ember-ship silhouette flying into it (or out of it — pick one, justify), city below, and the logo treatment from M5. Provide a written storyboard of the exact frame: element positions, glow intensities, whether particles are rising; define the `?shot=cover` staging variant if the current `start` shot can't carry it.
2. **Shot list & order** — 4–6 screenshots max. For each: the intended viewer takeaway, the seed, the moment (e.g. "frame where muzzle flash + 2 explosions + popup all coexist"), and which file name it should keep/replace.
3. **Staging hooks** — if a shot needs a bespoke moment (boss with 1/4 health for drama, victory with all 9 lanterns lit), specify the `?shot=` name and the deterministic autopilot script for it.
4. **Thumbnail test** — for each proposed shot, a short paragraph describing what survives at 480 px and what must be removed if it doesn't.
5. **Do not touch** `.makefun/owner.json`; screenshots must remain 1280×720 JPEG < 262144 bytes each (the cap enforced by main.ts and the submission validator), with margin to spare.

## Acceptance criteria

- Proposal includes an actual annotated ASCII frame diagram for the cover (rough layout is fine).
- The staging plan is executable by the implementing agent without design guesswork.
- If accepted, the four `screenshots/*.jpg` are replaced and `shots.json` updated in the same PR, followed by `MAKEFUN_OUTPUT_DIR=out node main.ts` + `makefun check` (0 failed) before submission.
