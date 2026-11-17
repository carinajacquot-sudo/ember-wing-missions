# STATUS — live board (append-only; do not delete history)

_Last updated by the director, 2026-11-17T03:05Z._

## Submissions

| # | Id | Claim code stored | Result | Notes |
|---|---|---|---|---|
| v1 | `01a1e72f-c06c-7761-9b36-13682bc08192` | vault (private) | Accepted 2026-11-17T01:47Z | **Contains the freeze bug** — superseded |
| v2 | `01a1e749-04c8-7027-96f1-084b953f960c` | vault (private) | Accepted 2026-11-17T02:14Z | Current best; fixed transform leak; re-shot screenshots |

- Both submissions are live entries in the dev heat; the final heat re-judges.
- Deadline for entries: **2026-11-18T18:00Z**. After that, no resubmission.
- Latest commit of the game repo: `e8a41ac` ("Re-shoot screenshots on the fixed build").

## Known issues (fixed vs open)

- ✅ FIXED (v2): `PowerupManager.draw()` unbalanced `save/restore` leaked the canvas transform — everything drawn after it (bullets, enemies, player, HUD) rendered off-screen; canvas froze mid-level-2 while the loop kept running. Root-caused via CDP playthrough + pixel-diff of frames; fixed by per-powerup save/translate/restore.
- ✅ FIXED: `out/` build artifacts were committed in `b4ea218`; removed and `out/` gitignored in the same fix commit.
- ⚠️ OPEN (cosmetic, accepted): sun is occluded by skyline with hard black edges in title/victory shots — deliberate synthwave slit-sun look, but could use a softer haze band.
- 💡 IDEAS (unscoped): title-screen ember drift, combo burn gauge, boss breakable core in phase 2, floating score popups, lantern HUD icons, boss health bar, skyline window color variance, neon rain. → see `missions/`.

## Decided

- Luna contributes **specs**, the implementing agent ships code in recorded sessions (see DIRECTIVES §3).
- No resubmission until the working tree passes: headless playthrough (60+ s, zero console errors, frames change), `?shot` screenshots re-taken, `MAKEFUN_OUTPUT_DIR=out node main.ts`, `makefun check` → 0 failed, push, submit.
- The cover stays `1-title.jpg` for now.

## Sandbox quirks (for the director, recurring)

- Local clones lose `.git` between sessions; binary installs (chromium, extracted Node 22) vanish. `/home/user` text files persist. The kit tarball + `node22.tar.xz` are the recovery anchors; GitHub is the source of truth.
- `~/.gemini` session logs can vanish; keep bundled transcripts backed up (done in the private vault).
