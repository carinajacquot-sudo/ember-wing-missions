# M4 — HUD redesign: every pixel carries a story

- **Target criteria**: completeness (1), polish (3) — **Priority P0** — Owner: Luna (spec) → implementing agent
- **Files**: `game/src/hud.js`, `game/src/assets/` (none — inline only), `game/src/config.js` (layout constants)

## Context

Current HUD is three text elements (lives, score+multiplier, level+lanterns). Legible but flat; judge shots will show it in every frame. Lives as text diamonds are weak; the boss shows no health; combo is a number, not a feeling.

## Deliverable (spec)

1. **Lives** — replace `♦` text with ember-flame glyphs: layered teardrop (outer orange, inner gold, white core), 18 px each; on life loss, the ember "gutters" (scale-pop + fade of the lost one, 400 ms).
2. **Combo gauge** — under the centered score: a 220 px segmented arc/bar that fills toward the next multiplier tier, burning gold→white when ×8 (pulsing); number remains but smaller, right of the bar.
3. **Score** — tabular numerals, subtle glow, thousands-separated (`12,650`).
4. **Lanterns** — replace `LANTERNS: 4/9` text with 9 small lantern icons in a row: unlit = dim outline, lit = warm glow. Level shown as roman-numeral-ish `LVL 2` prefix in cyan.
5. **Boss health bar** — during boss: thin full-width bar top-center under the score, violet with gold notches per 25 %; hidden otherwise (zero cost in non-boss shots).
6. **Wave banner** — on level start: "DISTRICT II — NEON RAIN" slides in from left with a low swell, 1.6 s, non-blocking.
7. **Low-HP state** — at 1 life, ember glyph pulses red and a soft red vignette breathes at the screen edges.
8. **Pause** — dim overlay + "PAUSED — press P" panel styled as a neon arcade cabinet card (rounded rect, double border, scanline fill).

## Acceptance criteria

- Every screenshot shows the new HUD; title screen shows logo only (HUD hidden there).
- HUD elements avoid the arena's central 60 % horizontally in the play shot (no occlusion of the ship/enemies).
- Icons drawn as canvas paths (no images); layout driven by constants in `config.js` (`hud` block: margins, sizes, bar width).
- Legible at 480 px short-side.
