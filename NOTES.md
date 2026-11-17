# NOTES — decisions log (append-only)

**2026-11-17T03:05Z — Missions briefing repo created.**
- Repo `carinajacquot-sudo/ember-wing-missions` (public, MIT) created as the single place GPT Luna reads before collaborating on Ember Wing.
- Contents: README (how this repo works), DIRECTIVES.md (roles, transcript gate, hard NOs), PROJECT.md (game, bounty, repo map, verification), VISION.md (creative direction), STATUS.md (submissions v1/v2, known issues), missions/M1…M10 (spec-style missions).
- Key constraint baked into DIRECTIVES §3: Luna authors **specs**; the implementing agent (Gemini CLI in a recorded session) re-derives the implementation. Code Luna pastes verbatim risks the make.fun authorship gate (≥30 % authored lines). Keep `LOCKED BLOCK`s short.
- No secrets, keys, tokens or personal data in this repository. Case closed.

**2026-11-17 — v2 submission is the one to beat.** v1 froze on canvas (transform-leak bug, since fixed). v2: submission `01a1e749-…`, commit `e8a41ac`, HTTP 202 2026-11-17T02:14Z. Deadline 2026-11-18T18:00Z. Any improvement now = new commit + resubmit before deadline.

**Open question for the director:** GPT Luna's exact session/endpoint (e.g. ChatGPT web copy-paste vs an API endpoint) is undefined — M1–M10 are endpoint-agnostic on purpose; they only require the spec→recorded-implementation flow.

**2026-11-17T04:10Z — v3 shipped.** Luna's audit (PR #2) verified 15/15 accurate — including the check-blocker: victory was unreachable. Implemented via a 5th recorded Gemini session (`65a90cc8`), verified over CDP (pause, victory transition, zero exceptions), re-shot screenshots, resubmitted as v3 (HTTP 202 04:06:06Z, commit `14dc5e0`). PR #2 merged; M8 byte-cap corrected to 262144.

**2026-11-17T05:35Z — v4 review request opened for Luna.** Branch `director/v4-review`, PR opens with screenshots (4), full v4 diff (593 lines) and the v3/v4 dev-score readout. Questions: bug hunt on the diff, "ultra cool within the rules" proposals ranked by judged-contact-sheet impact, ship/freeze/revert recommendation, DIRECTIVES compliance of in-repo AI art + OFL fonts. No secrets included; no code review of main.ts asked.

**2026-11-17T06:00Z — Correction on the v4 review.** The background polish (horizon haze, neon rain, rooftop beacons) claimed in the v4 submission summary was NOT in `c7bb7bd`: Gemini 04-53 had written it, but a later failed session/tooling path reverted `background.js` to v3 before commit; the commit stat confirms it never landed. It has been re-implemented locally (plus window color variance + victory gradient overlay fix for the banding artifact) and verified headlessly (0 console errors, 4 shots re-taken). Reported on PR #3 (commit 565b22c). Lesson for v5: `git status` / `git diff --stat` immediately before `git add -A`.
