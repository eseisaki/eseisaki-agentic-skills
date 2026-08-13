# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The user's personal, growing collection of Claude Code Agent Skills (`skills/<name>/SKILL.md`) for software engineering and productivity, plus the tooling that keeps them structurally sound and correctly triggered as the collection grows. There is no application code — the "product" is the skill files themselves, and this repo's own conventions for authoring them live in `docs/skill-anatomy.md` and `docs/plain-language-iso24495.md`.

## Commands

Pure Node, zero dependencies, no `package.json`/build step.

```bash
node scripts/validate-skills.js              # Tier 1: structural lint (frontmatter, sections, naming)
node scripts/validate-triggers.js             # Tier 2: trigger-routing check (positive/negative prompts, description collisions)
node scripts/validate-triggers.js --min-rank1 80   # Tier 2 with a stricter routing-quality floor
```

There is no automated Tier 3 (behavioral) runner — those evals are run live, by hand or via a subagent (see "Adding a skill" below).

To check a single skill, run the full validators above; both walk all of `skills/` and `evals/cases/` in one pass (there's no single-skill flag).

## Architecture

### The three-tier eval system

This is the load-bearing structure of the repo — see `evals/README.md` for full detail:

1. **Structural** (`scripts/validate-skills.js`) — checks every `skills/*/SKILL.md` against the rules in `docs/skill-anatomy.md`. The rules themselves live in `scripts/lib/skill-lint.js` as a single, unit-testable source of truth; `validate-skills.js` is a thin CLI wrapper over it.
2. **Trigger routing** (`scripts/validate-triggers.js`) — a zero-dependency, stemmed TF-IDF check that a skill's description carries the vocabulary users actually use and doesn't collide with another skill's description. It reads `evals/cases/<skill-name>.json` for each skill's `trigger.positive`/`trigger.negative` prompts and asserts they rank/don't rank the skill as expected.
3. **Behavioral** (no script) — an agent actually follows the drafted `SKILL.md` against each case file's `evals[].prompt` and a human or subagent judges the result against `expectations[]`. Intentionally manual; see `evals/README.md` for why (not worth automating at this pack's current size).

Every skill directory under `skills/` must have a matching `evals/cases/<skill-name>.json`, and vice versa — `validate-triggers.js` checks this coverage.

### Skill file structure

Each skill is self-contained under `skills/<skill-name>/`:

```
skills/<skill-name>/
  SKILL.md            # Required. Frontmatter (name, description) + standard sections
  scripts/            # Optional: runnable helpers, only if the skill needs them
  references/         # Optional: skill-specific reference docs
```

Key rules from `docs/skill-anatomy.md` (read that file for the full contract, don't duplicate it here):

- `name` in frontmatter must match the directory name; `description` must state both **what** the skill does and **when** to use it (a "Use when ..." clause), since that's the primary signal agents use to activate the skill and what Tier 2 checks mechanically.
- Checklists/reference material shared by *more than one* skill live in the repo-root `references/` directory, not inside any single skill's directory — colocating would force copying or a fragile "owning skill" convention. Material used by exactly one skill stays inside that skill's own directory.
- Keep `SKILL.md` under ~500 lines; only the name + description load at startup, the full file loads on demand, so heavier material belongs in supporting files loaded progressively.

### Adding a skill

Draft `SKILL.md` per `docs/skill-anatomy.md`, write `evals/cases/<skill-name>.json` per `evals/README.md`, run the behavioral evals live, then run both validators; iterate until both are clean and the behavioral evals pass. Skills are not considered done just because they look right on the page — the behavioral run and the deterministic validators are both required checks, not optional polish. `skills/create-skill/SKILL.md` is one skill in this collection (a meta-skill) that automates this exact flow — reach for it when the user wants to author or restructure a skill, but it's not the only thing this repo is for.

### Prose style

Skill prose (and anything meant to be read by an agent mid-task) follows `docs/plain-language-iso24495.md`: short sentences, active voice, common words over jargon, most-important-thing-first, headings/lists over walls of prose.
