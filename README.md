# eseisaki Agent Skills

My personal collection of [Claude Code Agent Skills](https://docs.claude.com/en/docs/claude-code) for software engineering and productivity — reusable workflows an agent can load on demand — plus the tooling that keeps them structurally sound and correctly triggered as the collection grows.

## Structure

```
skills/<skill-name>/SKILL.md   # One skill per directory; SKILL.md is the only required file
evals/cases/<skill-name>.json  # Trigger + behavioral test cases, one file per skill
references/                    # Checklists/docs shared across multiple skills
docs/skill-anatomy.md          # The format every SKILL.md follows
docs/plain-language-iso24495.md# Prose style guide for skill authoring
scripts/                       # Zero-dependency Node validators
```

## Validating skills

No dependencies to install — plain Node.

```bash
node scripts/validate-skills.js     # Tier 1: structure (frontmatter, sections, naming)
node scripts/validate-triggers.js   # Tier 2: trigger routing + description collisions
```

See `evals/README.md` for what each tier checks and the eval case file format.

## Adding a skill

Write `skills/<skill-name>/SKILL.md` following `docs/skill-anatomy.md`, add a matching `evals/cases/<skill-name>.json` per `evals/README.md`, then run the two validators above. The `create-skill` skill in this repo automates that whole flow inside Claude Code if you'd rather not do it by hand.
