# M9 — Code outline polish (how the source reads to the judge)

- **Target criteria**: structure (5) — **Priority P1** — Owner: Luna (review + spec) → implementing agent
- **Files**: game repo, whole `game/src/` (renames only with extreme care)

## Context

The judge receives file names, sizes and the first declarations of each source file. Our split is already decent (config/input/audio/background/player/enemies/bullets/powerups/particles/levels/hud/main/util). Structure is a free criterion — but names like `util.js` waste it.

## Deliverable (spec)

1. **Outline read-through** — simulate the judge: produce the exact outline the grader would show (list files + first 25 lines each), and critique it as a first-time reader.
2. **Header discipline** — spec a one-line banner comment for each `game/src/*.js` in the form: `// <Noun> — <its one job>. See VISION.md for tone.` (no credits-by-name, no dates).
3. **Rename candidates** (zero-risk, mechanical): `util.js` → `mathx.js`? `bullets.js` → `projectiles.js`? Only if the spec shows the rename end-to-end (imports updated, `?shot` unaffected, build byte-identical dossier). If risky, say no.
4. **First-declaration polish** — ensure the first declaration in each file is a meaningful named constant/class, not a helper.
5. **Anti-goals**: no re-architecture, no asset pipelines, no build step changes, no size inflation.

## Acceptance criteria

- The spec enumerates every file with its banner line and its intended "first impression" declaration.
- Rename list (if any) includes exact sed commands and the post-rename `?shot=start` smoke test.
- Total line delta ≤ 120 lines; dossier diff limited to nothing outside the touched files.
