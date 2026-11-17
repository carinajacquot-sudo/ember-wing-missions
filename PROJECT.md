# PROJECT — Ember Wing

## 1. The game

**Ember Wing** *(the last ember of a dying star)* — an original neon synthwave arcade shooter.

- The sun has died. You are the last ember, flying through the dead neon city **Cendre**, to relight the **nine district lanterns** before the Gloom swallows them.
- You play the **Ember** against the dead city's machines: drifters (pink darts), weavers (cyan sine-wave sentinels), tanks (orange hover-hulks), and the final boss, **The Hollow Crown** — a crowned void-skull pulsing radial, aimed, then bullet-hell patterns.
- 3 levels, 9 lanterns, 3 lives, a combo multiplier (×1…×8), powerups from tanks: **Flare** (triple shot), **Aegis** (shield), **Magnet** (pickup attraction + score bonus).
- Session flow: title → how-to → play → level-clear → victory (all 9 lanterns) / game over → restart. Pause with P, mute with M.
- Everything is procedural: canvas-drawn art, WebAudio-synthesized SFX and music, zero assets, zero network.

## 2. The bounty (make.fun — "Make the best game", $1,000)

- Harness `program/v1`, deadline **2026-11-18T18:00Z**. Entry = public GitHub repo + commit + recorded transcripts, judged by an AI comparing entries pairwise, top-100 to playoffs, final re-judged.
- **The judge sees**: one JPEG contact sheet of the screenshots (first = cover) + a short outline of the source (paths, sizes, first declarations). **It never plays.**
- Judging criteria, in order: (1) complete game shown in screenshots (start, HUD, win/lose/restart), (2) fun/depth, (3) feel/polish, (4) look/originality, (5) structure shown by the outline.
- House favor: strong, readable, complete visuals at glance speed; clean split of files with one job each; no minified/vendored code as source.

### How our entry is submitted

- Repository: https://github.com/carinajacquot-sudo/best-game-01 — snapshot taken at submission; later pushes do not change judged files.
- Two submissions exist: **v1** (01a1e72f…, contained a canvas-freeze bug — superseded) and **v2** (01a1e749-04c8-…, HTTP 202 at 2026-11-17T02:14Z, commit `e8a41ac`) which fixed it and re-shot all screenshots.
- Any new improvement ships as: change → headless verify → new commit → `makefun check` → `makefun submit` (a new submission; the last one before the deadline is what counts).
- `makefun check` must print **0 failed** before every submission (`transcripts_present`, `thinking_*`, snapshot caps, LICENSE SPDX line, README badges, `.makefun/owner.json`, topics, About→make.fun).

## 3. The repository layout

```
best-game-01/
├─ main.ts                # Node 22 entry: copies game/+screenshots/ to /output, writes dossier.json (buildDossier) — DO NOT touch semantics
├─ vendor/makefun-game.ts # make.fun's library, byte-identical — untouchable
├─ serve.ts               # dev static server (node serve.ts)
├─ game/
│  ├─ index.html          # container page, #gameCanvas 1280×720
│  └─ src/
│     ├─ main.js          # Game class: boot, state machine, loop, collisions, ?shot staging
│     ├─ config.js        # palette, sizes, tuning constants
│     ├─ input.js         # keyboard + mouse
│     ├─ audio.js         # WebAudio synth (SFX + 100 BPM minor bass/arp, mute M)
│     ├─ background.js    # parallax skyline, sun, stars, grid floor
│     ├─ player.js        # the Ember ship
│     ├─ enemies.js       # drifter/weaver/tank + Hollow Crown boss (3 phases)
│     ├─ bullets.js       # projectiles
│     ├─ powerups.js      # Flare / Aegis / Magnet (note: recently fixed transform leak)
│     ├─ particles.js     # ember trails, explosions, sparks
│     ├─ levels.js        # LevelManager: waves, lanterns, level flow
│     ├─ hud.js           # HUD + title/howto/levelclear/gameover/victory screens
│     └─ util.js          # seeded RNG, clamp, circleCollide, …
├─ screenshots/           # 4 JPEGs, 1280×720: 1-title, 2-play, 3-boss, 4-victory (cover = 1-title)
├─ shots.json             # names the ?shot= staging scenes
└─ LICENSE (MIT), README.md, .makefun/owner.json
```

## 4. Verification (nothing ships unverified)

- Serve `game/` and screenshot headlessly, e.g. `chromium --headless=new --window-size=1280,720 --virtual-time-budget=6000 --screenshot=out.png "http://127.0.0.1:8899/index.html?shot=play&seed=7"`.
- Staging scenes via `?shot=start|play|boss|end&seed=N` (deterministic autopilot, sets `window.__shotReady`).
- Full playthrough check: autopilot-injected scratch copy driven by CDP; zero console errors; canvas must differ frame-to-frame (guards against render freezes like the v1 bug).
- `node main.ts` with `MAKEFUN_OUTPUT_DIR=out` regenerates `out/dossier.json`; the checker requires it matches byte-for-byte.
- The `?shot` staging lines and the autopilot scratch copy are tooling; they don't ship in what the grader runs.

## 5. What the AI judge weighs (in order)

1. **Completeness shown in screenshots**: title, in-game HUD, win/lose screens, restart — all legible at thumbnail size.
2. **Fun/depth**: visible structure — levels, combo, boss phases, powerups.
3. **Feel/polish**: particles, glow, shake, popups, audio cues.
4. **Look/originality**: the Cendre city, sun, palette, typography coherence.
5. **Structure**: file split by job, plainly-named files/functions; the outline is the only code the judge reads — names must sell the design.

Every mission in `missions/` maps to one or more of these criteria; the header of each mission names its target criterion.
