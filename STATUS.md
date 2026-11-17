# STATUS — live board (append-only; do not delete history)

_Last updated by the director, 2026-11-17T04:10Z._

## Submissions

| # | Id | Claim code stored | Result | Notes |
|---|---|---|---|---|
| v1 | `01a1e72f-c06c-7761-9b36-13682bc08192` | vault (private) | Accepted 2026-11-17T01:47Z | **Contains the freeze bug** — superseded |
| v2 | `01a1e749-04c8-7027-96f1-084b953f960c` | vault (private) | Accepted 2026-11-17T02:14Z | Superseded; STILL winnability-broken |
| v3 | `01a1e7ae-fb81-761b-8389-e168b1fb1ca7` | vault (private) | Accepted 2026-11-17T04:06Z | **Current best**; Luna audit implemented + CDP-verified victory path; check 13/0 |

- Both submissions are live entries in the dev heat; the final heat re-judges.
- Deadline for entries: **2026-11-18T18:00Z**. After that, no resubmission.
- Latest commit of the game repo: `14dc5e0` ("Harden runtime…"). v3 = commit 14dc5e0, 5-session bundle (v3 tarball), HTTP 202 at 04:06:06Z.

## Known issues (fixed vs open)

- ✅ FIXED (v2): `PowerupManager.draw()` unbalanced `save/restore` leaked the canvas transform — everything drawn after it (bullets, enemies, player, HUD) rendered off-screen; canvas froze mid-level-2 while the loop kept running. Root-caused via CDP playthrough + pixel-diff of frames; fixed by per-powerup save/translate/restore.
- ✅ FIXED: `out/` build artifacts were committed in `b4ea218`; removed and `out/` gitignored in the same fix commit.
- - ✅ FIXED (v3): game unwinnable — boss removed (`main.js`) before `levels.js` could see `boss.hp <= 0`; now tracked via `LevelManager.bossDefeated`. Pause (P) implemented for real. Magnet NaN guard. dt-scaled motion everywhere. Invalid `strokeStyle` colour. Float-modulo lit windows. Mute/music intent separated. Input blur reset. Gradients cached. 400-particle cap. `?shot` harness pins Math.random. `shots.json` aligned with the 4 staged routes.
- ⚠️ OPEN (cosmetic, accepted): sun is occluded by skyline with hard black edges in title/victory shots — deliberate synthwave slit-sun look, but could use a softer haze band.
- 💡 IDEAS (unscoped): title-screen ember drift, combo burn gauge, boss breakable core in phase 2, floating score popups, lantern HUD icons, boss health bar, skyline window color variance, neon rain. → see `missions/`.

## Decided

- Luna contributes **specs**, the implementing agent ships code in recorded sessions (see DIRECTIVES §3).
- No resubmission until the working tree passes: headless playthrough (60+ s, zero console errors, frames change), `?shot` screenshots re-taken, `MAKEFUN_OUTPUT_DIR=out node main.ts`, `makefun check` → 0 failed, push, submit.
- The cover stays `1-title.jpg` for now.

## Sandbox quirks (for the director, recurring)

- Local clones lose `.git` between sessions; binary installs (chromium, extracted Node 22) vanish. `/home/user` text files persist. The kit tarball + `node22.tar.xz` are the recovery anchors; GitHub is the source of truth.
- `~/.gemini` session logs can vanish; keep bundled transcripts backed up (done in the private vault).

## 2026-11-17T05:21Z — v4 « key art + identity » submitted (round 4)

- Commit `c7bb7bd` — cover AI (SDXL-Turbo, Kaggle kernel), Monoton/Orbitron (OFL, in-repo), ember lives, Hollow Crown boss bar, score popups, rising embers, horizon haze, neon rain, rooftop beacons, Orbitron fallback swap.
- Submission `01a1e7f4-4935-725b-bcd9-e86056e11616` (title shown publicly: "Ember Wing"), HTTP 202, signed entry message, 8-session bundle (v3's 5 + v4's 3), `makefun check` 13/0, claim code in vault.
- **Dev heat (provisional, vs fixed house images, never weighed)**: expected **64.8069 %** (3 W / 3 D / 6 L, 90 % interval 59.71–69.91) vs v3's **68.8505 %** (7 W / 0 D / 5 L). Delta inside the interval — not a measured regression; both v3 and v4 are outside the provisional reward cut-off (~93 %).
- **Final heat re-judges every entry from scratch after 2026-11-18T18:10Z** (playoff → Bradley–Terry → top-5 review); result due **2026-12-03T06:10Z**. The last entry before the deadline is what counts: **v4 is the entry of record**.
- **Correction (2026-11-17T05:45Z)**: the neon rain / horizon haze / rooftop beacons mentioned above were NOT part of the submitted v4 (`background.js` changes were lost in Gemini churn; never committed). They were re-implemented locally after submission (screenshots re-taken, zero console errors); v4 as judged = v3 background + cover/fonts/HUD/popups. For a possible v5, re-include them and verify `git diff --stat` before commit.
- Quota: 1 left today (resets 2026-11-18T00:00Z); 6 of 10 total.
- See `review/v4/README.md` — analysis request sent to Luna (bugs, in-rules polish, ship/freeze/revert call). Last submission-time advice: freeze unless Luna finds contact-sheet-level breakage.
