# M6 — Depth mechanics: burn gauge & the Hollow Crown core

- **Target criteria**: fun/depth (2) — **Priority P1** — Owner: Luna (spec) → implementing agent
- **Files**: `game/src/main.js`, `game/src/levels.js`, `game/src/enemies.js`, `game/src/hud.js`, `game/src/config.js`

## Context

Combo resets on hit — good — but there's no commitment ladder. The boss has 3 movement phases but no vulnerability rhythm; a bullet-sponge mid-fight drags the judge playthrough (not that the judge plays — but the human shows matter at the end, and the recorded session must not look broken).

## Deliverable (spec)

Two additive systems, both optional-safe (game must remain beatable if both disabled via config flags):

### A. Burn Gauge (combo made physical)

- A 5-segment ember bar under the combo bar: fills with hits, drains slowly (6 %/s). Full bar = **Ignition** (8 s): ship invincible-looking (hot aura), fire rate ×1.5, bullets pierce, screen edges glow gold. Taking a hit clears the gauge instead of a life (only if gauge ≥ 60 %) — rewards aggression without extra difficulty.
- Audio: low rumble swell on full, whoosh on ignition.
- HUD: bar sits under combo bar (from M4); label appears only at full ("IGNITION").

### B. The Hollow Crown — shield & core

- Phase 2: the boss grows a **cracked void shield** — a ring segment (270° of circle) that absorbs damage; every 8th hit the ring cracks visually (adds a jagged stroke layer); after 5 cracks the shield shatters (burst + 2 s stagger + 5× damage window on the body).
- Phase 3: core exposed — the crown's center eye becomes a weak point; shots into the core (radius test, generous 18 px) deal double damage and pop tiny gold sparks + higher-pitch hit sfx.
- Phase transitions: 1 s invulnerable shimmer + shockwave + "THE CROWN SMILES" text flash (once per fight).

## Acceptance criteria

- Flags `config.gameplay.burnGauge` and `config.gameplay.bossCore` default **true**; disabling either yields a coherent game.
- Boss fight duration target: 75–120 s for a competent player.
- All numbers above tunable from config; spec lists recommended values + one "easier" and one "harder" alternate dial.
- Headless playthrough completes to victory with flags on, zero console errors.
