# M1 — Visual overhaul: sky, sun, city

- **Target criteria**: look/originality (4), screenshot wow (1) — **Priority P0** — Owner: Luna (spec) → implementing agent
- **Files**: `game/src/background.js`, `game/src/config.js` (palette only)

## Context

The horizon axis is sacred (see VISION.md §Composition rules). Current layers: gradient sky, 80 stars, sun with scanline cuts, 3 skyline layers, perspective grid floor, no moon/planets, flat building faces, uniform window speckle.

## Deliverable (spec, not code)

A concrete spec covering, each with exact numbers, draw-order and palette refs:

1. **Sun upgrade** — layered radial glow + an inner hot core + 2–3 slow vertical "ember bands" drifting upward inside the disc; slit-sun scanlines kept but softer (alpha 0.6 max); a wide additive haze band at the horizon behind the skyline (kills the hard black rectangle look).
2. **Sky** — subtle aurora curtain (2 crossed sine ribbons, additive, very low alpha) + a second, smaller dead sun / ringed gas giant low in the west sky (one accent, not two).
3. **City** — 4th depth layer (near-black silhouette at the bottom corners, framing); window lights in 3 warm hues (amber/rose/teal) with per-building hue drift; a few blinking rooftop beacons (CSS-free, sin(time)); antenna silhouettes on random towers.
4. **Neon rain** — 40–60 thin vertical streaks, additive low alpha, falling with slight slant, only in upper 60 % of sky; must not clutter the HUD zone.
5. **Grid floor** — keep perspective grid; add a faint horizontal pulse traveling outward from horizon (energy ring every ~4 s).
6. **Performance budget**: ≤ 140 extra draw ops/frame, no per-frame allocations, no gradients created per frame (cache them at construction).

## Acceptance criteria

- Title and play screenshots remain readable at 480 px short-side thumbnail (verify by downscaling).
- 60 fps at 1280×720 with entities + particles active (autopilot smoke run, zero console errors).
- No new palette colors outside VISION.md's table (extensions of existing hues OK).
- Sun still anchored at (640, 320), radius 160.
- Spec fits one PR; include one "cover alternative" composition idea for the title shot.
