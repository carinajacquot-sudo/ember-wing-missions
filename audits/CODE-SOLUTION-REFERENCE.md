# Code solution reference — Ember Wing

> **Status: reference proposals, not a tested patch.** These examples are intended for the implementing agent to re-derive against the current game files, add characterization tests, and validate in its recorded session. They deliberately do not claim to compile unchanged: method names, call order, collision flow, and state ownership must be checked against the implementation before applying. Follow `DIRECTIVES.md`: Luna supplies design/code guidance; the implementing agent authors and tests the game changes.

## 1. Boss defeat must survive boss removal

### Suggested state model

Use a persistent level/result flag for progression and remove the dead entity from combat immediately. Reset the flag on a new run. Ensure death rewards/effects run once, not once per frame.

```js
// LevelManager constructor and reset
this.bossDefeated = false;
```

At the lethal collision site, guard the one-shot transition before clearing the object:

```js
const boss = this.enemies.boss;
if (boss && boss.hp <= 0 && !this.levels.bossDefeated) {
  this.levels.bossDefeated = true;
  this.score += boss.scoreValue;
  this.particles.explosion(boss.x, boss.y, CONFIG.colors.violet, 60);
  this.audio.playBoom();
  this.enemies.boss = null;
}
```

Adapt `score`, `particles`, `audio`, and `CONFIG` references to the existing class structure; do not duplicate existing reward logic. In level progression, consume the state instead of requiring a still-existing boss:

```js
if (this.currentLevel === 3 && this.bossDefeated) {
  this.lanternsLit = this.targetLanterns * this.maxLevels;
  this.levelCompleted = true;
}
```

If `lanternsLit` already carries previous level progress, use the project’s established total rather than multiplying blindly. The key invariant is that the terminal event remains observable after the entity is removed.

### Required tests

- Lethal hit awards score and emits effects exactly once.
- Boss is removed and cannot attack/collide after death.
- Level 3 completes, all lanterns reach 9, and victory appears.
- Restart resets `bossDefeated` and does not instantly win.

## 2. Make gameplay motion consistent across frame rates

First categorize every value as per-frame displacement, units/second, or seconds. To preserve the existing 60 Hz tuning, convert legacy per-update displacement with `dt * 60`. Do this for all moving systems in one coordinated change, not just the player.

```js
// Example inside a player update(dx, dy, dt)
const frameScale = dt * 60;
this.x += dx * this.speed * frameScale;
this.y += dy * this.speed * frameScale;
```

Use the same convention for bullets, enemies, boss, particles, stars and powerups. Clamp the accumulated frame delta after tab suspension to prevent a giant teleport:

```js
const dt = Math.min((now - this.lastTime) / 1000, 0.05);
this.lastTime = now;
```

If a random event historically had probability `p` per 60 Hz update, make its probability equivalent for a delta `dt`:

```js
const probability = 1 - Math.pow(1 - p, dt * 60);
if (rng.next() < probability) {
  // perform event
}
```

Prefer the seeded RNG already used by the game. Timers already expressed in seconds should remain seconds and should not receive an extra factor of 60. Check all `update()` signatures and call sites before editing.

### Required tests

Simulate one second at 30, 60, and 120 updates; compare travel distance, cooldown count and expected random-event frequency within a reasonable tolerance. Test tab-resume delta clamping and gameplay balance.

## 3. Make Magnet attraction safe at zero distance

The current normalization risks division by zero when pickup and player overlap. Avoid normalizing tiny vectors, and avoid stepping past the target:

```js
const dx = player.x - p.x;
const dy = player.y - p.y;
const distSq = dx * dx + dy * dy;

if (player.magnetTimer > 0 && distSq > 1 && distSq < 300 * 300) {
  const dist = Math.sqrt(distSq);
  const step = Math.min(6, dist); // convert to dt-scaled units with the motion pass
  p.x += (dx / dist) * step;
  p.y += (dy / dist) * step;
}
```

Do not compute `dx / dist` when `dist` is zero or nearly zero. If attraction is changed to a continuous speed, use `step = Math.min(MAGNET_SPEED * dt, dist)`.

### Required tests

Exact overlap, near-zero distance, distance just below/above 300, Magnet inactive, pickup collection, and finite (`Number.isFinite`) coordinates after every update.

## 4. Fix Canvas background color and stable window placement

Remove the semicolon from the color string (Canvas expects a CSS color, not a CSS declaration):

```js
ctx.strokeStyle = 'rgba(0, 240, 255, 0.25)';
```

Avoid modulo on floating-point world coordinates for window selection. Use integer cell indices and a stable building index:

```js
for (let row = 0, wy = y + 15; wy < 540; row++, wy += 30) {
  for (let col = 0, wx = b.x + 10; wx < b.x + b.w - 10; col++, wx += 20) {
    if ((col + row * 3 + buildingIndex) % 4 === 0) {
      ctx.fillRect(wx, wy, 8, 12);
    }
  }
}
```

Adapt boundaries to actual building geometry; ensure `buildingIndex` comes from the existing outer iteration without per-frame allocation. Check the screenshot before/after; visual output may intentionally change.

## 5. Implement pause as a simulation state

A pause must stop world updates while continuing to render a clear overlay and accept the resume key. Do not let `dt` accumulate while paused. One safe pattern is to process the edge-triggered key before world updates:

```js
if (input.wasPressed('KeyP') && this.state === 'playing') {
  this.paused = !this.paused;
  this.lastTime = performance.now();
}

if (!this.paused) {
  this.updateWorld(dt);
}
this.draw();
if (this.paused) this.drawPauseOverlay();
```

Use the actual game-state names and ensure `wasPressed()` is consumed exactly once. Define behavior on title, victory, game over and focus loss. Avoid adding separate competing pause flags if the state machine can represent pause cleanly.

### Required tests

While paused, player/enemy/bullet positions and timers do not advance; P resumes once; a long pause does not create a huge simulation step; title/end screens remain correct.

## 6. Separate mute from music intent

Mute is an output preference; it should not itself decide whether music should be running. Represent those concepts separately:

```js
constructor() {
  this.muted = false;
  this.musicRequested = false;
  this.musicTimer = null;
}

startMusic() {
  this.musicRequested = true;
  if (this.muted || this.musicTimer) return;
  // create exactly one timer/context schedule
}

stopMusic() {
  this.musicRequested = false;
  if (this.musicTimer) clearInterval(this.musicTimer);
  this.musicTimer = null;
}

toggleMute() {
  this.muted = !this.muted;
  if (this.muted && this.musicTimer) {
    clearInterval(this.musicTimer);
    this.musicTimer = null;
  } else if (!this.muted && this.musicRequested) {
    this.startMusic();
  }
  return this.muted;
}
```

In a real implementation, avoid `startMusic()` re-setting `musicRequested` in a way that creates an accidental restart; separate `requestMusic()` from the low-level scheduler if that makes state clearer. Stop/request music explicitly on game-state transitions. Resume AudioContext only after a user gesture as required by browser policy.

### Required tests

Unmute after explicit stop does not restart; unmute during requested playback resumes once; no duplicate intervals; transitions to victory/game over stop music as designed.

## 7. Input focus and canvas bounds

On window blur or document becoming hidden, clear held input so a missed keyup cannot leave movement/fire stuck:

```js
this._onBlur = () => {
  this.keys.clear();
  this.pressed.clear();
  this.mouse.down = false;
};
window.addEventListener('blur', this._onBlur);
```

Remove the listener in `destroy()`. For mouse input, attach handlers to the canvas/container and convert client coordinates using that element’s bounding rectangle; do not let clicking unrelated page regions start firing. If supporting touch is a goal, implement and test pointer/touch behavior explicitly; otherwise correct the input module comment so it does not claim unsupported controls.

## 8. Deterministic screenshot harness

Seed all randomness used by staged scenes, including background construction, enemies and particles. Avoid `Math.random()` in capture paths. Prefer injecting an RNG or deterministic scene data over globally replacing `Math.random()`, which can perturb unrelated code. Render the same route and seed twice and compare image hashes. Keep regular gameplay behavior intact.

Align `shots.json` only with routes that the harness really implements. `main.ts` sets a screenshot cap of 262144 bytes per image, so validate the actual byte count below that limit, not merely below 350 KB. Preserve the existing `window.__shotReady` handshake and verify all routes finish without console errors.

## Implementation protocol

1. Re-read the current implementation and tests in the game repository; these examples were composed from static inspection and may need adaptation.
2. Add characterization tests before each behavior change.
3. Re-derive the patch in the recorded implementing-agent session; do not claim this reference was implemented.
4. Keep changes narrow, run relevant tests and lint/type checks if configured.
5. Run the project-required 60+ second headless playthrough, screenshot harness, `MAKEFUN_OUTPUT_DIR=out node main.ts`, and `makefun check` with zero failures before submission.
6. Report command output, screenshots’ dimensions/byte sizes and any unavailable checks. Do not mark unexecuted validation as passed.
