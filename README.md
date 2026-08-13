# eseisaki Agent Skills

A pack of [Claude Code Agent Skills](https://docs.claude.com/en/docs/claude-code) — reusable workflows an agent can load on demand — plus the tooling that keeps them structurally sound and correctly triggered.

## Structure

```md
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

Use the `create-skill` skill (`skills/create-skill/SKILL.md`) inside Claude Code — it walks through capturing intent, drafting the `SKILL.md`, writing the eval case file, running the behavioral evals live, and validating the result. See `docs/skill-anatomy.md` for the file format if you're writing one by hand.
