# Ember Wing — Mission Briefing for Collaborators

**Read this file first, then `DIRECTIVES.md`, then `PROJECT.md`.**

## What this repository is

This is the complete onboarding and mission briefing for any collaborator working on **Ember Wing**, an original neon synthwave arcade shooter, submitted to [make.fun](https://make.fun/bounty/best-game-01)'s **"Make the best game"** bounty ($1,000, deadline 2026-11-18T18:00Z).

- The game itself lives in **https://github.com/carinajacquot-sudo/best-game-01** (public, MIT). All code lands there — never in this repository.
- This repository holds **vision, directives, missions, status and reference notes**. No secrets, no keys, no wallets, no personal data — ever. If you think you need one, stop and ask the maintainer.
- The coding agent who implements here is bound by the competition's rules (see `DIRECTIVES.md` §3): code must be authored in recorded agent sessions, so your contribution is a **specification**, not paste-ready source.

## How to work (the 30-second version)

1. Read, in order: `DIRECTIVES.md` → `PROJECT.md` → `VISION.md` → `STATUS.md`.
2. Pick a mission from `missions/` (priority order P0 → P1 → P2).
3. Produce the mission's **Deliverable** (a spec), in the file path the mission names, on a branch `luna/<mission-name>`, and open a PR.
4. Never mark a mission complete until its **Acceptance criteria** are all checkable.
5. Ask before: touching submission state, changing scope, committing binaries, or adding dependencies to the game.

## Ground rules (non-negotiable)

- **No secrets in either repository.** Not API keys, not tokens, not `.env` files, not personal data. The game repository's CI checks the whole history.
- **No code you cannot explain.** You may propose implementations, but the implementing agent re-derives them in a recorded session; wholesale pasted code risks the competition's authorship checks.
- **No edits to `main` without a PR.** Branch → PR → the maintainer (or the implementing agent) merges.
- **Be honest.** If a mission is blocked or a proposal is speculation, say so explicitly in the PR.
- **Deadline is real:** 2026-11-18T18:00Z. Missions are ordered by impact on the AI judge (see `PROJECT.md` §5).

## File map

| File | Purpose |
|---|---|
| `DIRECTIVES.md` | The director's orders — hard constraints, roles, quality bar |
| `PROJECT.md` | The game, the bounty, the judging, the repository, the tech |
| `VISION.md` | Creative direction: tone, palette, typography, references |
| `STATUS.md` | Live status: submissions, known bugs, what's decided |
| `missions/M*.md` | One file per mission: context, deliverable, acceptance criteria |
| `NOTES.md` | Meeting notes / decisions log (append-only) |
