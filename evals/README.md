# Skill Evals

How this repo checks that a skill triggers when it should and does what it says. Adapted from [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)'s three-tier eval system, trimmed to what's worth maintaining while this pack is small — see `docs/skill-anatomy.md` for the structural rules these evals sit on top of, and the `create-skill` skill for the workflow that produces them.

## The tiers

| Tier | What it checks | How it runs |
|---|---|---|
| 1. Structural | Frontmatter, naming, required sections | `node scripts/validate-skills.js` |
| 2. Trigger routing | Positive prompts rank their skill top-k; negative prompts don't; no two descriptions collide | `node scripts/validate-triggers.js` |
| 3. Behavioral | An agent following the skill satisfies its `expectations[]` | Run live, by hand or via a subagent — no automated grader |

Tiers 1 and 2 are deterministic, zero-dependency Node scripts, ported from Osmani's originals. Tier 3 is intentionally **not** automated here: the original's behavioral runner spins up throwaway git workspaces, materializes fixtures, and grades a `claude -p` execution trace with a second `claude -p` call — real infra worth the cost once a pack has many skills and needs CI-safe regression checks, but overkill for a small, actively-authored one. Instead, `create-skill` has you (or a subagent) run each behavioral eval prompt directly against the drafted skill and judge the `expectations[]` yourself, using the case file as the record of what was checked.

Tier 2 is a lexical approximation (stemmed TF-IDF over descriptions) — it catches a description missing the vocabulary users actually say, or one so broad it outranks the skill that should own a prompt. It cannot judge semantics; that's what Tier 3 is for. Note it's also close to a no-op with only one or two skills in the catalog — there's nothing to collide with yet — but it costs nothing to run and starts paying off as the pack grows.

## Eval case format

One file per skill: `evals/cases/<skill-name>.json`.

```json
{
  "skill_name": "example-skill",
  "trigger": {
    "positive": [
      { "prompt": "Realistic thing a user would actually type", "top_k": 3 }
    ],
    "negative": [
      { "prompt": "A prompt that belongs to a different skill", "owner": "other-skill-name" }
    ]
  },
  "evals": [
    {
      "id": 1,
      "prompt": "The task prompt to run against the skill",
      "expected_output": "Description of what a good result looks like",
      "expectations": [
        "A concrete, checkable behavior the run should exhibit"
      ]
    }
  ]
}
```

- `trigger.positive` — at least 3. Realistic prompts a user would actually say; `top_k` defaults to 3 (tighten to 1 for the skill's signature ask). Paraphrase how users talk — copying the description just games the check.
- `trigger.negative` — at least 2. Prompts that belong to a *different* skill. Declare `owner` when you can name that skill; the check then asserts the owner outranks this skill for the prompt, turning it into a real pairwise test instead of one that passes vacuously when the prompt matches nothing.
- `evals` — at least 1. No `files[]`/fixtures required (that's Tier 3 execution-runner machinery this repo doesn't use). Each `expectations[]` entry should be a specific, checkable behavior — not a vague quality bar — since a human or subagent has to judge it directly against the run.

## Running

```bash
node scripts/validate-skills.js
node scripts/validate-triggers.js
node scripts/validate-triggers.js --min-rank1 80   # optional routing-quality floor
```
