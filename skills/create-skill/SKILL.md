---
name: create-skill
description: Guides authoring a new agent skill end to end — a short intent interview, drafting the SKILL.md, writing a lightweight eval case file, running the behavioral evals live, and validating structure and trigger routing. Use when the user wants to create a new skill, turn a workflow or set of instructions into a skill, write or restructure a SKILL.md, or improve an existing skill's description so it triggers correctly.
---

# Create Skill

## Overview

Turns a workflow, a set of instructions, or a vague idea into a working `SKILL.md`: confirm intent, draft the skill, write a small set of test prompts, run them for real, and validate the result. Every step produces something checkable — a confirmed intent, a structured draft, a passing test run, a clean validator — so a skill is only called done when it has actually been shown to work, not just written.

## When to Use

- The user wants to create a new skill from scratch, or turn an existing workflow/prompt/set of instructions into one.
- The user wants to restructure or clean up an existing `SKILL.md` to match this repo's conventions.
- The user's skill isn't triggering reliably and the description needs work.
- NOT for one-off task automation that will only ever run once — skills are for workflows worth reusing. If in doubt, ask.
- NOT for heavy iterative optimization across many trial runs and quantitative benchmarks — use a dedicated benchmarking workflow for that.

## Core Process

### 1. Capture intent

Before drafting anything, get clear, concise answers to:

1. What should this skill enable an agent to do?
2. When should it trigger — what phrases or situations should cause an agent to reach for it?
3. What does a good outcome look like (expected output/deliverable)?
4. Are there edge cases or exclusions worth calling out ("NOT for X")?

If the user already described the workflow in this conversation, extract answers from that instead of re-asking. Keep this short — a few targeted questions, not a full requirements interview. Confirm your understanding before moving on.

### 2. Draft the SKILL.md

Create `skills/<skill-name>/SKILL.md` following the structure in `docs/skill-anatomy.md` (frontmatter contract, standard sections, naming conventions). Don't duplicate that document here — read it if you need the details.

Points worth restating because they're easy to under-weight:

- The `description` must state both **what** the skill does and **when** to use it — a bare "Use when …" clause is required, since that's the primary triggering signal and also what Tier 2 below checks mechanically.
- Only add `scripts/`, `references/`, or other supporting files if the skill actually needs them.
- Write in imperative, process-oriented language — steps an agent follows, not background knowledge.
- Hold the prose to `docs/plain-language-iso24495.md`: short sentences, active voice, common words over jargon, most-important-thing-first, headings/lists over walls of prose. Run its "Practical checklist for agent output" against the draft before moving on — a skill an agent has to re-read twice to follow is a drafting bug, not a style nitpick.

### 3. Write the eval case file

Create `evals/cases/<skill-name>.json` following the format in `evals/README.md`: `trigger.positive` (≥3 realistic prompts that should route here), `trigger.negative` (≥2 prompts that belong to a different skill, with `owner` where you can name it), and `evals` (≥1 behavioral case with a prompt, `expected_output`, and concrete `expectations[]`).

Paraphrase how a real user would actually ask — don't copy phrases straight out of the description you just wrote, that games the routing check instead of testing it.

### 4. Run the behavioral eval(s) live

For each entry in `evals[]`, actually perform the task: either do it yourself following the drafted `SKILL.md`, or spawn a subagent with the skill and the eval prompt. Compare the result against `expectations[]` one by one and report which passed, which didn't, and why. This is a judgment call you make directly, not a step to skip.

### 5. Validate

Run both deterministic checks and fix anything they flag before considering the skill done:

```bash
node scripts/validate-skills.js
node scripts/validate-triggers.js
```

The first checks structure (frontmatter, sections, naming). The second checks that the positive/negative trigger prompts actually route to this skill and that the description doesn't collide with another skill's.

### 6. Iterate

If the behavioral eval or the user's own read of the draft surfaces problems, revise the `SKILL.md`, then re-run steps 4 and 5. Stop when the behavioral evals pass, the validators are clean, and the user is satisfied — not after a fixed number of rounds.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The use case is obvious, skip the interview" | A skill drafted from an assumed intent tends to be vague on exactly the trigger conditions and edge cases that matter most — the few questions are cheap insurance. |
| "I'll just describe the skill in the description, the eval case file is busywork" | Without positive/negative trigger prompts, an over-broad or under-specific description won't get caught until it silently mis-triggers (or fails to trigger) in real use. The case file is the only place that's checked. |
| "The trigger prompts can just restate the description" | That games Tier 2 instead of testing it — it'll rank perfectly and still fail to trigger on how users actually phrase the ask. Paraphrase realistically. |
| "I read the draft, it looks right, no need to actually run it" | Reading a skill and running it surface different failures — steps that seem clear on the page are often ambiguous or out of order once an agent tries to follow them literally. |
| "The validators are just style nitpicks, ship it anyway" | `validate-skills.js` errors mean the skill is structurally broken for other tooling that parses `SKILL.md` (frontmatter, section anatomy); `validate-triggers.js` errors mean it likely won't fire, or will fire on the wrong prompts, in real use. |
| "The reader is an agent, not a person, so plain language doesn't apply" | The agent following this skill parses it the same way a rushed human reader would: linearly, once, under time pressure. Dense jargon or a 40-word sentence costs it the same ambiguity it would cost a person. |

## Red Flags

- Drafting `SKILL.md` before confirming what it should do and when it should trigger.
- A description with no "Use when" clause, or one so generic it could apply to several different tasks.
- Trigger prompts in the case file that are near-identical to the description text.
- Skipping step 4 and asserting the skill "should work" without having run it.
- Ignoring `validate-skills.js` or `validate-triggers.js` errors instead of fixing them.
- Building out a large benchmarking setup (parallel trial runs, a review dashboard, automated description tuning) for a single small skill — that's disproportionate; use it only if the user explicitly wants that level of rigor.

## Verification

- [ ] `skills/<skill-name>/SKILL.md` exists with valid frontmatter (`name` matches the directory, `description` states what and when)
- [ ] All required sections from `docs/skill-anatomy.md` are present
- [ ] The draft passes the `docs/plain-language-iso24495.md` checklist (short sentences, active voice, no unexplained jargon, important-first, scannable structure)
- [ ] `evals/cases/<skill-name>.json` exists with ≥3 positive triggers, ≥2 negative triggers, ≥1 behavioral eval
- [ ] Each behavioral eval was actually run and its `expectations[]` checked against the real result
- [ ] `node scripts/validate-skills.js` passes with 0 errors
- [ ] `node scripts/validate-triggers.js` passes with 0 errors
- [ ] The user has confirmed the drafted skill matches what they wanted
