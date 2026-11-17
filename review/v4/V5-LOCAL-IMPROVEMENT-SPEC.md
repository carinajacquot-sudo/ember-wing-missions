# V5 local refinement — implementation reference and precise code sketches

**Target:** improve the local Ember Wing build for review only. **Do not submit v5 until the director gives a new order.**

- Game source reviewed: `carinajacquot-sudo/best-game-01` at v4 commit `c7bb7bd4019af619fc48918ae37b7aed2086fd1e`.
- Collaboration branch: `director/v4-review`.
- This is a code-level implementation reference for the recorded implementing agent, not a patch applied to the game. The examples below must be adapted to the current source and re-derived in the recorded session per `DIRECTIVES.md` §3.
- No local browser, screenshot comparison, game test, or build was run while writing this reference. Treat visual diagnosis as a testable hypothesis.

## Recommendation and scope

Keep v4 as the current submitted version. Work on a local v5 branch without submitting. Start with the high-confidence capture synchronization defect. Then isolate the victory background if a pixel/visual comparison confirms that world grid or scanline patterns are bleeding through the overlay. Add deterministic harness construction if repeated captures differ. Do not add a new gameplay feature in this pass; it raises risk without addressing the known screenshot reliability issue.

Suggested local sequence:

1. Add harness tests/recorded reproduction before changing the path.
2. Make screenshot readiness await required assets and final paint.
3. Make the victory background opaque before layering cover art and text.
4. Make seeded scene construction actually consume the requested seed.
5. Compare all four screenshots at native and 480 px height; run existing project checks.
6. Keep changes local. Do not submit until explicitly asked.

## A. Fix the screenshot readiness race

### Current issue

`Game.init()` constructs `Background` before it reads `?seed=`, and `stageScreenshot()` sets `window.__shotReady = true` after two animation frames, while `document.fonts.load()` runs asynchronously afterward. The cover `Image` is also initiated in the constructor without awaiting decode. Therefore the capture harness may treat an unready canvas as final. This is a code-confirmed ordering flaw; whether it caused a particular submitted JPEG to use fallback content must be reproduced.

### 1. Promise-based image loader

Add a small helper in `main.js` or a dedicated existing asset utility (avoid a new dependency):

```js
function loadImage(url) {
  return new Promise((resolve, reject) => {
    const image = new Image();
    image.onload = async () => {
      try {
        if (typeof image.decode === 'function') await image.decode();
        resolve(image);
      } catch (error) {
        reject(new Error(`Image decode failed: ${url}`));
      }
    };
    image.onerror = () => reject(new Error(`Image load failed: ${url}`));
    image.src = url;
  });
}
```

Resolve only once; load errors must reject instead of silently yielding a screenshot with missing art. `onload` plus `decode()` may report an already-decoded image, which is fine. If the supported browser requires a compatibility fallback, implement and test it rather than setting readiness on a timeout.

### 2. Share the decoded cover between Game and HUD

At present the game and HUD both create their own `Image` for the same cover. Use one decoded asset and pass it to the HUD, so the readiness check covers exactly the image that is drawn.

In `HUD`:

```js
export class HUD {
  constructor(cover = null) {
    this.cover = cover;
  }

  setCover(cover) {
    this.cover = cover;
  }

  // In drawTitleScreen/drawVictory, draw this.cover when usable.
  // Do not create another Image inside each draw method.
}
```

In `Game`, construct the HUD with no image, then, during async harness preparation, set the decoded object using `this.hud.setCover(this.cover)`. For ordinary play, either await the same asset once before starting the loop or retain an explicit legacy fallback until the image resolves and request a redraw when it does. Keep a deliberate fallback for normal play if the asset is unavailable; for screenshot mode, fail the shot rather than quietly claiming a complete capture.

### 3. Wait for the exact font faces used by staged screens

Use `document.fonts.load()` before the final screenshot paint. Include the actual families/styles/text used in title and victory draw paths. A minimal focused preload for the screen headlines is:

```js
async function loadScreenshotFonts() {
  if (!document.fonts?.load) {
    throw new Error('FontFaceSet.load is unavailable in screenshot browser');
  }

  const requests = [
    ['112px Monoton', 'EMBER'],
    ['54px Monoton', 'W I N G'],
    ['50px Monoton', 'THE CITY REMEMBERS'],
    ['15px Orbitron', 'ALL 9 LANTERNS RELIGHTED'],
    ['700 34px Orbitron', '12850'],
    ['600 19px Orbitron', '[ ENTER ] REIGNITE'],
  ];

  await Promise.all(requests.map(([font, text]) => document.fonts.load(font, text)));
  await document.fonts.ready;

  for (const [font, text] of requests) {
    if (!document.fonts.check(font, text)) {
      throw new Error(`Required screenshot font unavailable: ${font}`);
    }
  }
}
```

Expand this list only for font requests actually used by capture routes. Do not consider `document.fonts.ready` alone proof that a requested family loaded: it can settle while fallback is used. If fonts fail, set a visible harness error and leave readiness false.

### 4. Make the harness initialization await resources and paint

Change only the screenshot branch to an awaited pipeline. Keep the normal game loop and state behavior unchanged. Factor the two-frame paint into a helper:

```js
function nextFrame() {
  return new Promise(resolve => requestAnimationFrame(() => resolve()));
}

async function waitForFinalPaint(render) {
  render();
  await nextFrame();
  render();
  await nextFrame();
}
```

Then use the following ordering in the screenshot path (adapt staging call/seed order to the actual class):

```js
async prepareScreenshot(shot) {
  try {
    await Promise.all([
      loadScreenshotFonts(),
      this.loadCoverForShot(),
    ]);

    this.hud.setCover(this.cover);
    this.stageScreenshot(shot);
    await waitForFinalPaint(() => this.render());

    window.__shotReady = true;
  } catch (error) {
    window.__shotReady = false;
    window.__shotError = String(error?.message ?? error);
    console.error('[screenshot harness]', error);
  }
}
```

`loadCoverForShot()` should assign the one decoded image to `this.cover`; it can simply await `loadImage('assets/ember-wing-cover.jpg')`. In the screenshot branch, initialize `window.__shotReady = false` and clear `window.__shotError` before starting. `init()` should call `void this.prepareScreenshot(shot)` and return without starting the normal RAF loop. Remove the earlier readiness assignment from `stageScreenshot()`; only the outer pipeline owns readiness.

Do not use a timer fallback that marks a failed image/font load ready. The capture runner must detect `window.__shotError` or fail on a bounded timeout. This yields a truthful failure instead of a subtly wrong JPEG.

## B. Isolate victory compositing from the live gameplay background

### Code-confirmed compositing risk

The main renderer draws the normal background before `HUD.drawVictory()`. The victory function paints the cover at alpha `0.52` and then paints a `rgba(..., 0.75)` veil. At those opacities, pixels from the already-drawn grid/scanlines can still contribute. The README reports horizontal banding in the victory capture, but the actual JPEG pixels were not inspected here; verify the source by comparing screenshots before deciding the final treatment.

### Target treatment

Make the victory panel own an opaque base first, so no gameplay world lines can bleed into it. Then draw the cover as a controlled, subdued layer and place the text over it. This keeps the game’s current overall composition while removing dependence on what `Background.draw()` drew underneath.

Replace the background portion of `drawVictory()` with this pattern, retaining the existing typography and text positions unless visual inspection calls for a specific adjustment:

```js
// Victory screen owns its background: cover opaque gameplay grid/scanlines first.
ctx.save();
ctx.globalAlpha = 1;
ctx.fillStyle = '#05020a';
ctx.fillRect(0, 0, 1280, 720);

if (this.cover && this.cover.complete && this.cover.naturalWidth > 0) {
  const scale = Math.max(
    1280 / this.cover.naturalWidth,
    720 / this.cover.naturalHeight,
  );
  const width = this.cover.naturalWidth * scale;
  const height = this.cover.naturalHeight * scale;
  const x = (1280 - width) / 2;
  const y = (720 - height) / 2;

  ctx.globalAlpha = 0.58;
  ctx.drawImage(this.cover, x, y, width, height);
  ctx.globalAlpha = 1;

  // The opaque base above prevents world layers showing through this veil.
  ctx.fillStyle = 'rgba(5, 2, 10, 0.34)';
  ctx.fillRect(0, 0, 1280, 720);
}

// Draw existing victory title, lantern line, score and restart prompt here.
// Use save/restore around text effects; restore this outer context at the end.
```

If the cover is required for the v4 visual identity, screenshot mode should fail when it did not load (section A), not silently use a different composition. For normal gameplay, provide an intentional solid-color fallback. Do not add a second gradient band behind only the wordmark unless screenshots prove it improves contrast; a localized gradient can itself look like the reported banding. If a gradient is truly needed, use a smooth, broad gradient and inspect its 8-bit JPEG result.

### Visual acceptance tests

- Capture v4 and local v5 with identical route and seed; compare the victory frame at 1280×720 and 480 px high.
- Confirm the wordmark, lantern line, final score and prompt remain legible; no HUD/world grid or sun scanlines cross the typography.
- Inspect for new posterization/banding introduced by the overlay itself. If the old artifact was in the source image rather than bleed-through, this change may not fix it; isolate that cause before further tuning.

## C. Make screenshot seeds control scene construction

### Ordering flaw

`Game` constructs `Background` in its constructor before `init()` reads the query string, and `Background` generates stars and skyline with `Math.random()` in its constructor. Replacing `Math.random` later in `init()` cannot retroactively seed those positions. Thus the `?seed=` value does not control all screenshot scene inputs.

### Preferred fix: pass a seeded RNG into constructors

Parse the seed before constructing randomized scene systems. Keep RNG local rather than replacing global `Math.random` (global replacement can mask accidental randomness and leak across code paths).

Example shape:

```js
function readSeed(search) {
  const raw = new URLSearchParams(search).get('seed');
  if (raw === null) return 12345;
  const parsed = Number.parseInt(raw, 10);
  return Number.isFinite(parsed) ? (parsed >>> 0) : 12345;
}

class Game {
  constructor(seed = 12345) {
    this.rng = new RNG(seed);
    this.background = new Background(this.rng);
    this.enemies = new EnemyManager(this.rng);
    this.particles = new ParticleManager(this.rng);
    // construct remaining systems as today
  }
}

window.addEventListener('DOMContentLoaded', () => {
  const game = new Game(readSeed(window.location.search));
  game.init();
});
```

Then update only randomized operations to consume the injected RNG: `Background` constructor and `generateSkyline`, enemy spawn-time variation and attacks if randomized, particle explosions, and any other capture-visible random placement. Ensure harness staging and normal play continue to call the same public methods. If full RNG injection is too broad for this local pass, deterministic capture fixtures can provide fixed background/entity/particle data explicitly; do not claim the seed is complete if only some objects use it.

Delete `Math.random = () => this.rng.next()` from screenshot init after all relevant randomness has been injected. A global override changes platform behavior and does not fix constructor-time randomness.

### Determinism checks

- Same shot route + seed, fresh page loads twice → identical pixel hashes after readiness.
- Different seeds → at least one intended procedural feature differs.
- No-seed normal play starts and behaves normally.
- No test mutates global `Math.random` after the page is initialized.
- Ensure `Date.now()` blinking or animation phase does not make staged title images nondeterministic. In capture mode, use a fixed phase/always-visible prompt; keep normal blinking for regular gameplay.

## D. One possible screenshot-safe title blink

The fallback title prompt uses `Date.now()` to blink. The cover path uses stable prompts, but if fallback is ever captured it can change between images. Add an explicit capture mode to HUD rather than altering normal title timing:

```js
// HUD constructor or setter
this.screenshotMode = false;

// In the fallback title screen
const blink = this.screenshotMode || Math.floor(Date.now() / 400) % 2 === 0;
```

Set `this.hud.screenshotMode = true` before staging a screenshot and reset it only if the harness object is reused for normal play. Prefer failing the screenshot on failed assets (section A), so the legacy fallback does not become the intended v5 title capture.

## E. Art and font policy gate — resolve before choosing v5 art work

`DIRECTIVES.md` §4 includes “All art procedural or inline SVG/canvas.” The v4 README says the cover is an in-repository AI-generated JPEG. Repository-local storage/no CDN does not by itself resolve that apparent policy conflict. Before spending v5 effort on new art, the maintainer/director must explicitly clarify whether the existing cover is an allowed exception. If not, implement a procedural/Canvas or inline SVG replacement and re-check screenshots; do not silently reinterpret the rule.

The fonts are local and accompanied by OFL text files. Preserve the license notices and verify the license conditions and font attribution/provenance in the repository. Do not fetch fonts at runtime.

## F. Test plan and do-not-submit guard

1. Add or run a harness test proving readiness remains false until image/font loading and the last paint finish.
2. Verify failed image/font loading reports `window.__shotError`, keeps readiness false, and does not hang the test suite indefinitely.
3. Re-run four screenshots twice with fixed seeds and compare hashes; inspect the victory frame for the specific banding cause.
4. Confirm browser console has no errors and every expected route eventually sets readiness true.
5. Check 1280×720, JPEG encoding, file sizes against the actual project gate, and contact sheet at 480 px high.
6. Run the project’s required headless playthrough and `makefun check`; record real command output. Do not change `main.ts` semantics or vendor code.
7. **Do not submit v5 or change the currently submitted entry.** Keep this work on the local development branch until the director provides a new order. The v4 submission remains operative meanwhile.

## Summary

The strongest v5 is not “more effects”; it is a trustworthy screenshot pipeline, isolated victory composition, and genuinely seeded staging. These changes target visible polish and evaluation reliability while keeping gameplay and the currently submitted v4 untouched until explicitly authorized.
