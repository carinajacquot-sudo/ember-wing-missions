# VISION — Ember Wing

## The one-liner

> The sun has died. Its last ember is flying into the dead neon city of Cendre to relight the nine district lanterns, before the Gloom swallows everything.

## Tone

Melancholic heroism. Not edgy, not parodic — a eulogy with a pulse. Every visual decision should feel like a machine that refuses to stop singing after the world ended. Synthwave as requiem: 1984 arcade cabinet meets city-at-duskelegy.

## Palette (reference `game/src/config.js`)

| Role | Hex | Use |
|---|---|---|
| Deep void | `#05020a` | sky top, vignette base |
| Twilight purple | `#150a2b → #2b0c3a` | sky gradient, city depth |
| Sunset magenta | `#ff007f` | sun core edge, drifters, boss eyes glow |
| Ember orange | `#ff4400` / `#ff7700` | sun mid, tanks, aegis shield, UI accents |
| Solar gold | `#ffea00` / `#ffee00` | sun center, score, combo, Flare, crown |
| Neon cyan | `#00f0ff` | player, weaver, bullets, HUD-primary |
| Void violet | `#9d00ff` / `#2a0d4a` | boss body, far skyline layer |
| Ghost white | `#ffffff` | cores, text highlights |

Rule: one dominant temperature per layer (hot center, cool edges). Never introduce a new accent without removing an old one.

## Typography

- All text: `Courier New` (monospace) — already the game's voice; keep it.
- Title: bold 84px, magenta glow, wide tracking feel (simulate with spaces if needed).
- HUD: bold 22px, slight glow; numbers always centered in their slot.
- Screens: at most three text levels per screen; everything else is art.

## Composition rules

1. **Horizon anchor**: the sun sits on the horizon at y≈320, radius ≈160; skyline layers stack below it (3 depths), grid floor below the horizon. Never break this axis — it's the game's identity.
2. **Negative space at the top**: the sky stays mostly empty so the HUD breathes (and screenshots read instantly).
3. **Pop against dark**: player is always the brightest cold object on screen; enemies are warm/hot or violet; the eye must find the Ember in < 200 ms.
4. **Glow is meaning**: glow = energy/damage/interaction. Background glow is free; entity glow is signal.
5. **Readability before spectacle**: if a shot looks "cool" at 100 % but unreadable at 25 % (thumbnail), it fails the judge. Prefer big shapes, few layers, strong value contrast.

## Motion grammar

- Player: ease-out banking (visual only), engine-flame flicker, ember trail particles that die behind it.
- Enemies: drifters dart-judder; weavers sine; tanks heavy drift + slow turret sweep; boss pendulum sway, crown glints per phase.
- Feedback: hit → spark + 3-frame white flash + tiny shake; kill → multi-stage explosion (flash, ring, debris, ember rain) + score popup; player hit → red flash, big shake, heartbeat audio duck.
- Camera: subtle 4–6 px shake on impacts; 10–14 px on player hit and boss death; never rotational.

## Audio identity (already WebAudio, extend, don't replace)

100 BPM minor loop: sawtooth bass line (A minor) + square arp + noise-hats. SFX layered: shoot = short square blip w/ pitch slide; hit = noise+tri thunk; boom = filtered noise burst + sub drop; pickup = major third chirp; UI = click; boss roar = detuned saw swell.

## What "truly impressive" means for the judge

- A cover (title) shot that sells the world in one glance: dying sun, dead city, one small light.
- A gameplay shot dense with intent: firefight, particles, popups, HUD full of signal — yet every element readable.
- A boss shot that promises scale: boss large in frame, bullet rings in flight, health bar, 6/9 lanterns.
- A victory shot with emotional release: lantern rays relighting the skyline + stats.

## The fantasy, restated for copy

Heroic last-stand: every pixel is either dying light or defiant light.
