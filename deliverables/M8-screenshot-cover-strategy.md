# M8 — Screenshot and cover strategy

- **Target criteria:** completeness (1), cover impact (1)
- **Priority:** P0
- **Scope:** capture plan and staging requirements for `carinajacquot-sudo/best-game-01`; this document does not claim the game, screenshots, or tests have been changed.
- **Owner:** Luna (spec) → implementing agent (recorded implementation) → maintainer (review and submission)

## Why this mission matters

The make.fun judge evaluates a contact sheet and a short source outline; it does not play the game. The first image is the cover. The four existing images should therefore read as one compact argument: **world → combat → boss → earned victory**. Do not trade legibility for visual density.

## Verified repository facts

Inspected the public game repository on 2026-11-17:

- `game/src/main.js` stages `start`, `play`, `boss`, and `end` when `shot` is present. The `boss` scene sets level 3, 6 lanterns, score 9800, spawns the boss, sets its HP to 22, launches two attacks, and paints two explosion bursts. The `end` scene sets victory, score 12850, and 9 lit lanterns.
- The current `shots.json` lists only `start`, `play`, and `end`; it omits the implemented `boss` staging scene.
- `screenshots/` currently contains `1-title.jpg`, `2-play.jpg`, `3-boss.jpg`, and `4-victory.jpg`.
- The game canvas is 1280×720. Mission requirement: each submitted screenshot should be JPEG, 1280×720, and below 350 KB.

These facts were checked from the repository files; no local browser run or image pixel inspection was available during this specification. Any visual judgments below are design targets, not claims that current images already meet them.

## Proposed final shot list and order

| Order / file | Staging | Viewer takeaway | Frame requirements |
|---|---|---|---|
| 1 — `screenshots/1-title.jpg` | New `?shot=cover&seed=7` | A tiny, defiant ember flies into a dead neon city beneath a dying sun. | Strong title silhouette; ship, sun, and skyline all recognizable at thumbnail size. |
| 2 — `screenshots/2-play.jpg` | Existing `?shot=play&seed=7` | The game has readable, varied enemies and active combat. | Ship unobscured; player shots and at least two explosion stages visible if the current harness reliably renders them; HUD legible. |
| 3 — `screenshots/3-boss.jpg` | Existing `?shot=boss&seed=7` | The Hollow Crown is the large, distinct final threat. | Boss and crown dominate the right-side arena; ship and representative bullets remain distinguishable; HUD does not collide with the boss. |
| 4 — `screenshots/4-victory.jpg` | Existing `?shot=end&seed=7` | The mission resolves: all nine lanterns are lit. | Victory headline, 9/9 progression, and final score readable against the skyline. |

Keep the established filenames and the four-image limit. Do not add a fifth image unless the maintainer explicitly changes the submission scope.

## Cover storyboard (`?shot=cover&seed=7`)

Canvas coordinates are 1280×720. Treat these as initial composition targets; the implementing agent should render, inspect, and adjust only within the constraints below.

```text
x=0                  320                 640                960             1280
┌──────────────────────────────────────────────────────────────────────────────┐ y=0
│                           EMBER                                              │
│                         W I N G                                              │
│             the last ember of a dying star                                  │
│                                                                              │
│                                . .                                           │
│                    ☄  ───────────►     (sun center: 640, 320; r=160)         │
│                                  \   dying sun                              │
│                                   \                                         │
│             skyline / horizon ─────\─────────────────────────────────────── │ y≈520
│        one restrained prompt: [ ENTER ] REIGNITE                             │
└──────────────────────────────────────────────────────────────────────────────┘ y=720
```

### Geometry and hierarchy

- Preserve the established sun anchor: center `(640, 320)`, radius `160 px`.
- Keep the skyline and ground/grid below the sun/horizon axis; do not move the world anchor to make room for text.
- Place the ship near `(455, 390)`, directed right toward the sun. Use the existing player silhouette; avoid inventing a new asset.
- Keep the title in the upper sky, with `EMBER` above `W I N G`; put the existing subtitle below the title and above the sun where it does not cover the disc.
- Show exactly one instruction line on the cover: `[ ENTER ] REIGNITE`. Omit how-to and mute prompts in this staged cover only. Normal title-screen controls remain unchanged.
- The cover scene is screenshot harness tooling, not a new gameplay state. It must render deterministically for the same seed and must not change normal title behavior.
- Do not claim that the existing title implementation already has this composition. `stageScreenshot()` currently supports `start`, not `cover`; implementation is required before capture.

### Contrast and effects

- The ship is the highest-contrast cool object; the sun remains the dominant warm mass. Preserve negative space around the logo.
- Keep ember particles in the lower third or between the ship and horizon; do not let them cross the logo or subtitle.
- Avoid placing the ship directly over the sun's brightest core: preserve a visible silhouette and clear direction of travel.
- At 480 px output height (0.667 scale), the ship becomes about 12 px long if based on the current 26 px arrow. Confirm it is still identifiable; if not, adjust only the staged ship scale or position, not the player entity's gameplay size.
- Do not add a moon/planet or extra accent to this cover. One focal celestial body is enough.

## Shot-specific staging guidance

### Play

The current `play` harness deliberately seeds a firefight: level 2, four lit lanterns, score 1450, combo 4, Flare active, two weavers, one tank, four drifters, five player bullets, and two explosion particle bursts. Preserve this as the baseline. Inspect the rendered frame before changing entity coordinates or counts. If the frame is cluttered, reduce simultaneous effects before removing enemy-type variety. Do not describe an unimplemented score popup or muzzle flash as present.

### Boss

Use the already implemented `boss` stage rather than inventing a second boss scene. Verify the staged HP and phase visual match the actual rendering: the current setup assigns 22/40 HP, while `EnemyManager.update()` changes phases at 15 HP and 5 HP. Consequently the initial stage should be expected to depict phase 1 unless the staging setup explicitly adjusts phase; do not label it phase 2 without visual/code verification. Confirm no offscreen or overlapping bullets undermine the composition.

### Victory

Use the existing `end` stage and verify its rendered HUD/screen treatment. The current `stageScreenshot('end')` sets state to `victory`, score to 12850, and lanterns to 9; it does not by itself prove that nine skyline lantern glows animate or that the score is visible. Inspect the actual capture and keep only truthful visual claims.

## Determinism and `shots.json`

After the cover staging exists, update `shots.json` to list the four final captures in contact-sheet order, with explicit deterministic seeds:

```json
[
  { "name": "1-title.jpg", "query": "?shot=cover&seed=7" },
  { "name": "2-play.jpg", "query": "?shot=play&seed=7" },
  { "name": "3-boss.jpg", "query": "?shot=boss&seed=7" },
  { "name": "4-victory.jpg", "query": "?shot=end&seed=7" }
]
```

This is the **target manifest**, not a claim that the game repo has already been updated. Validate that the screenshot tool honors the manifest order and waits for `window.__shotReady`. Preserve the existing `start`, `play`, `boss`, and `end` harness behavior unless a narrow, tested change is required.

## Acceptance and verification checklist

The implementing agent should complete and record all applicable checks before asking for review:

1. Add the `cover` staging branch in `stageScreenshot()`; leave the regular title and gameplay flow untouched.
2. Render all four manifest entries twice with the same seeds; verify each pair is pixel-identical (or document any unavoidable nondeterministic source and remove it from staged rendering).
3. Inspect all frames at native size and at 480 px height. Each shot must communicate its listed takeaway; HUD and title copy must not be clipped or obscured.
4. Confirm every image is JPEG, exactly 1280×720, and strictly below 350 KB.
5. Confirm all four staged routes set `window.__shotReady`, produce no browser console errors, and do not hang.
6. Run the mission-required playthrough and repository submission checks. In particular, project status requires a 60+ second headless playthrough, no console errors, changing frames, `MAKEFUN_OUTPUT_DIR=out node main.ts`, and `makefun check` with 0 failures before submission.
7. Re-shoot all four images in the same PR if any screenshot is changed; do not replace or rewrite `screenshots/` without re-shooting and verifying the whole set.
8. Report actual command output, image dimensions/sizes, and any failed or unavailable check. Do not mark the mission complete on the basis of this spec alone.

## Scope and safety boundaries

- One mission per PR; target game-repository PR title: `M8: screenshot and cover strategy`.
- The implementation must be authored by the recorded implementing agent, not pasted wholesale from this specification.
- Do not edit `.makefun/owner.json`; do not introduce dependencies, external assets, network requests, or new palette accents.
- Screenshots are binary outputs and must be generated from the reviewed build, never hand-composited to imply gameplay that does not exist.
- This deliverable belongs in the mission/briefing repository only. It does not alter or claim changes to the game repository.

## Review status

**Specification ready; implementation and screenshot verification pending.** The boss-defeat state transition is a separate suspected gameplay blocker found during static review; validate and address it through its own tested change before resubmission. M8 must not be represented as verified until the checklist above has been completed.
