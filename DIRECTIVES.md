# DIRECTIVES — the director's orders

These directives are binding on every collaborator, including GPT Luna. They exist because Ember Wing is entered in a judged competition with strict entry rules.

## 1. Roles

- **The maintainer** (carinajacquot-sudo): owns both repositories, the make.fun account, all keys and decisions, final merge authority.
- **GPT Luna** (collaborator): deep-design brain. Responsibilities: creative direction, game design, mechanics design, balancing, copywriting, spec writing, review of screenshots and playability. Luna does **not** implement code in the game repository and does **not** commit to it directly.
- **The implementing agent** (Gemini CLI, recorded sessions): writes all game code inside recorded sessions, under the competition's transcript rules.
- **The director** (the operator running the agent): sequences work, verifies, submits, talks to both of you.

## 2. What everyone optimizes for

1. **The AI judge's eyes**: the judge sees one contact sheet of screenshots + a short outline of the source (file names, first declarations). It never plays the game. Screenshots must show a complete loop at a glance.
2. **Robustness**: the entry's `main.ts` and `game/` will run in the grader. A frozen canvas loses everything. Never ship untested render or update paths.
3. **Honesty**: this is the maintainer's own project with the maintainer's own AI collaborators. No passing off others' work as own; the transcripts show the real process.

## 3. The transcript gate (why Luna writes specs, not shipped code)

make.fun's gate requires ≥ 30 % of the entry's lines to be authored inside our recorded agent sessions, ≤ 40 % "unwritten" lines (absent, or read in via import commands), ≥ 5 assistant turns, ≥ 2 files revised, usage priced 100 %. Lines Luna proposes are fine when the implementing agent **re-derives them** (typing them into the session itself). Lines Luna emits that are verbatim-pasted by a human are "pasted" lines: they count as not-authored but they are NOT "unwritten" as long as they appear in the session transcript (e.g. in the human's message). Numerically safe up to a few hundred lines — but the cleanest path, and the one we mandate, is:

> **Luna produces a design spec. The implementing agent implements it.**

If Luna genuinely believes a specific literal code block is required (exact constant, exact SVG path, exact color regex), mark it as `LOCKED BLOCK` in the spec and keep it short.

## 4. Quality bar

- **Visible polish beats hidden complexity.** The judge cannot see your architecture; it can see glow, composition, motion, readability.
- **Every change is verifiable.** A mission is not done because it "sounds done"; it is done when the acceptance criteria in its file are demonstrably met (a screenshot, a score, a measured frame).
- **Performance is part of feel**: hold 60 fps at 1280×720 with ~50 entities and particles on screen; no per-frame allocations in hot loops; `shadowBlur` used deliberately, not everywhere.
- **No frameworks, no network, no assets outside the repo.** Plain canvas + WebAudio. All art procedural or inline SVG/canvas.
- **Originality.** No ripping of assets or layouts from other games. Ember Wing's identity is its own: ember/phoenix/city-of-cinders/synthwave-elegy.

## 5. Branching & process

- Branch names: `luna/M1-visual-overhaul`, `luna/M4-hud-redesign`, …
- One mission per PR; PR title `M<id>: <short>`; PR body lists which acceptance criteria are addressed and how.
- Small commits with plain messages; never rewrite published history; never force-push `main`.
- After merge, the implementing agent re-tests headlessly before any resubmission.

## 6. Hard NOs

- No secrets, credentials, tokens, `.env`, or personal data in any repository, issue, PR, comment or log.
- No `.makefun/owner.json` edits by anyone but the implementing agent under maintainer direction.
- No changes to `main.ts` semantics (it must keep calling `buildDossier` from `vendor/makefun-game.ts`, unchanged vendor file).
- No new dependencies; no CDN links; no external fonts/images.
- No deleting or rewriting `screenshots/` without re-shooting afterwards in the same PR.
- No posting strategy or status of *other* competitions or accounts anywhere public.

## 7. Escalation

If a mission is ambiguous, contradicts itself, or you are unsure about scope: do not guess — leave a note in the PR marked `DIRECTOR QUESTION:` with the exact question, and stop that mission.
