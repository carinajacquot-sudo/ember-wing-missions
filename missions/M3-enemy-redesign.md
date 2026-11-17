# M3 — Enemy redesign: silhouettes with intent

- **Target criteria**: look/originality (4), fun/depth (2) — **Priority P1** — Owner: Luna (spec incl. inline SVG/canvas path data) → implementing agent
- **Files**: `game/src/enemies.js`, `game/src/config.js` (palette)

## Context

Current sprites: drifter = magenta chevron, weaver = cyan circle+stroke, tank = orange rounded square, boss = violet disc + yellow crown + red eyes. Distinctive but flat; nothing communicates "machine of the dead sun".

## Deliverable (spec)

Design each enemy as a **procedural canvas path spec** (moveTo/lineTo/arc coordinates, closed shapes, at least 2 sub-shapes + 1 accent each), plus animation notes:

1. **Drifter** — "shard wasp": angular dart body split into 3 armor plates with 1-frame wing flicker every ~0.5 s; hot pink hull, white cockpit slit, twin forward fangs. Hints kamikaze.
2. **Weaver** — "ribbon serpent": 3 segmented rings linked in a sine chain (head + 2 echo ghosts at −20 px / −40 px on the path, fading); cyan hull, white eyes on stalks. Reads as graceful, alien, sine-motion at a glance.
3. **Tank** — "bulwark hover-hulk": wide hull trapezoid + side skirts, glowing orange viewport slit, 2 riveted seams, slow turret nub on top tracking player x; on death, drops powerup with a casket-Open animation (simple scale-in).
4. **The Hollow Crown** (boss) — redrawn at 1.6× size: obsidian skull (low-poly arcs), gold crown with 3 spikes that glint in sequence per phase, eyes that shift hue per phase (phase 1 magenta, 2 ember orange, 3 white-hot); phase-transition burst ring + invulnerable 1 s shimmer; phase 2 gains a cracked shield ring (see M6 core-shield).
5. **Readability check**: every enemy must be identifiable in silhouette at 32 px on dark background.
6. Draw budget: ≤ 1.5× current per-enemy path cost.

## Acceptance criteria

- Boss screenshot shows the crown glint + phase-2 aura; play screenshot shows drifter/weaver/tank all on-screen and distinguishable.
- Silhouette test documented (the spec includes a 32 px render matrix of each enemy, ASCII or described).
- No image assets; all canvas paths.
