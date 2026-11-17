# M2 — Combat feel: the Ember must feel alive

- **Target criteria**: feel/polish (3), fun/depth (2) — **Priority P0** — Owner: Luna (spec) → implementing agent
- **Files**: `game/src/player.js`, `game/src/particles.js`, `game/src/bullets.js`, `game/src/main.js` (FX hooks only), `game/src/config.js` (tuning)

## Context

The ship is a 26 px arrow with a white core. Firing is silent visually; kills pop but lack layering; no screen shake except structure; score popup doesn't exist. Judge shot 2 (play) is the single most-scrutinized frame after the cover — it must feel like an explosion already happened and another is imminent.

## Deliverable (spec)

1. **Engine flame** — two particle emitters at the wing roots (left/right): ember sparks, orange→gold→transparent, spawn rate scales with speed input; 1.5 px in radius, life 0.4 s; additive blend.
2. **Thruster bloom** — a soft radial glow behind the ship at low alpha, pulsating (sin(t·8)).
3. **Muzzle flash** — 60 ms star burst at the barrel on fire + a faint cyan recoil nudge on the sprite (2 px, spring back).
4. **Ship hit flash** — 120 ms white overlay on the ship silhouette (globalCompositeOperation 'source-atop' or pre-rendered white silhouette), plus magenta spark burst, plus 8 px shake for 180 ms.
5. **Multi-stage explosion** (on any enemy death): stage A flash disc (80 ms), stage B shockwave ring expanding 300 px (stroke, additive, fade), stage C 20–30 debris particles with gravity-lite fall, stage D 8 ember motes that linger and drift (3 s).
6. **Screen shake system** — `fx.shake(amp, ms)`; amp: drifter kill 2, weaver kill 3, tank kill 5, player hit 9, boss phase break 12, boss death 14. Exponential decay. Camera applies offset to the whole world render (wrap world draw in translate), never to HUD.
7. **Floating score popups** — on kill: "+100 ×3" etc., gold, rises 40 px over 0.8 s, fades out; crit kills (combo ≥ 6) get larger text + extra shake.
8. **Bullet streaks** — player bullets gain a short motion streak (previous-frame ghost at 40 % alpha); enemy bullets get a hot rim (thin white arc on the colored disc).
9. **Performance**: pre-allocate particle pool (cap 400), no string concat in hot paths.

## Acceptance criteria

- Play screenshot shows: flame trail, muzzle flash mid-frame, ≥ 2 active explosions in different stages, ≥ 1 popup, HUD unobstructed.
- No frame drops below 55 fps in the autopilot run with 8 enemies + 30 particles.
- Shake never exceeds 14 px and never rotates the canvas.
- All FX tunables live in `config.js` (new `fx` block).
