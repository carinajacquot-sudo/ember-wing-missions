# NOTES — decisions log (append-only)

**2026-11-17T03:05Z — Missions briefing repo created.**
- Repo `carinajacquot-sudo/ember-wing-missions` (public, MIT) created as the single place GPT Luna reads before collaborating on Ember Wing.
- Contents: README (how this repo works), DIRECTIVES.md (roles, transcript gate, hard NOs), PROJECT.md (game, bounty, repo map, verification), VISION.md (creative direction), STATUS.md (submissions v1/v2, known issues), missions/M1…M10 (spec-style missions).
- Key constraint baked into DIRECTIVES §3: Luna authors **specs**; the implementing agent (Gemini CLI in a recorded session) re-derives the implementation. Code Luna pastes verbatim risks the make.fun authorship gate (≥30 % authored lines). Keep `LOCKED BLOCK`s short.
- No secrets, keys, tokens or personal data in this repository. Case closed.

**2026-11-17 — v2 submission is the one to beat.** v1 froze on canvas (transform-leak bug, since fixed). v2: submission `01a1e749-…`, commit `e8a41ac`, HTTP 202 2026-11-17T02:14Z. Deadline 2026-11-18T18:00Z. Any improvement now = new commit + resubmit before deadline.

**Open question for the director:** GPT Luna's exact session/endpoint (e.g. ChatGPT web copy-paste vs an API endpoint) is undefined — M1–M10 are endpoint-agnostic on purpose; they only require the spec→recorded-implementation flow.
