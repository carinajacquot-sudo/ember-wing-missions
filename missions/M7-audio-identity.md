# M7 — Audio identity: music states & ducking

- **Target criteria**: feel/polish (3) — **Priority P2** — Owner: Luna (spec) → implementing agent
- **Files**: `game/src/audio.js`, `game/src/main.js` (state hooks), `game/src/config.js`

## Context

Music is a single A-minor 100 BPM loop all game. SFX exist but overlap unducked at loud moments. Good enough; not memorable.

## Deliverable (spec)

1. **Three music states** (same core loop, layered stems): `explore` (bass + hats only), `combat` (+ arp, + snare on 2 & 4), `boss` (+ detuned fifth pad, + tom heartbeat at 50 BPM double-time, tempo 100→112 feel via added layers, not real tempo change). Crossfade 800 ms on state change.
2. **Ducking** — any explosion SFX ducks music −6 dB for 400 ms (compressor-sidechain feel via gain node automation); player-hit ducks −9 dB / 600 ms.
3. **Ignition fanfare** (if M6 lands) — a one-shot 4-note minor-to-major rise on Ignition activation.
4. **Victory motif** — 2 bars, same progression resolved to major; plays once over the victory skyline relight.
5. **UI ticks** — menu move, confirm, deny as distinct short blips.
6. All samplers stay procedural (oscillators/noise buffers); no files; total new code ≤ ~150 lines; every state object listed with notes/gain/filter/ADSR in the spec.

## Acceptance criteria

- Muting with M silences everything (music+SFX) — no leaked nodes (verify by calling toggleMute twice and measuring master gain).
- No clipping at Ignition (peak ≤ −1 dB per the analyser).
- Boss state audibly distinct on a 5-second blind test (director listens).
