# M5 — Screens: title, how-to, level-clear, victory, game over

- **Target criteria**: completeness (1), look/originality (4) — **Priority P0** — Owner: Luna (spec, incl. copy) → implementing agent
- **Files**: `game/src/hud.js` (screen functions), `game/src/main.js` (state transitions only if needed)

## Context

Screens work but are static text slabs. The title is the judge's cover — it must carry the fantasy in one glance and animate subtly in live play.

## Deliverable (spec)

1. **Title** — composition: dying sun low, dead-city skyline, one ember logo.
   - Logo "EMBER" (magenta neon, flicker ±5 % alpha every 2–4 s) over "W I N G" (cyan outline, tracking-spaced). Subtitle lowercased cyan "the last ember of a dying star".
   - 12–18 tiny ember particles drift upward from the bottom third (additive, random respawn), all in the title state only.
   - Prompt block restyled: `[ ENTER ] REIGNITE` / `[ H ] HOW TO FLY` / `[ M ] SOUND ON/OFF` — one line each, dimmed key in brackets, bright action text; a hairline neon divider above.
   - High-score line appears if `localStorage.best > 0` ("BEST FLIGHT — 12,650").
2. **How to play** — neon terminal panel: scanline-textured translucent card, header `FLIGHT MANUAL // EMBER WING`, rows with key caps drawn as rounded-rect chips (`↑↓←→` / `WASD`, `SPACE`, `P`, `M`), mission text: relight 9 lanterns across 3 districts; watch for the Hollow Crown. Footer prompt identical styling to title return.
3. **Level clear** — radial burst from screen center (additive rays), "DISTRICT RELIT" big, sub "Lanterns 6/9 restored to the city", stats line (time, accuracy proxy, bonus), press-enter pulse.
4. **Victory** — the skyline relights: 9 lantern glows bloom across the city silhouette one by one (time-staggered), sun swells warmer, "THE CITY REMEMBERS" headline, final score + best combo + "PRESS [ENTER] TO FLY AGAIN".
5. **Game over** — ember dims to a spark and falls; "THE GLOOM TAKES CENDRE" (replaces GAME OVER), final score, "PRESS [ENTER] TO REIGNITE".
6. **Copy discipline**: max 1 exclamatory line per screen; no marketing voice; all-caps for headlines, sentence case for body.

## Acceptance criteria

- Title at 480 px thumbnail still shows: logo, sun, city, one prompt line — verified by downscaled render.
- All five screens reachable and exit via the documented keys; zero console errors; each screen's copy matches this spec verbatim unless the PR proposes a change with rationale.
- `window.__shotReady` staging (?shot=start|play|end) still works untouched.
