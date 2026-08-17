---
name: organize
description: Runs the user's personal PARA/GTD idea-organizing workflow — capturing raw ideas, filing them into Projects/Areas/Resources/Archive, scoring Projects on Impact/Effort, listing the current Impact/Effort ranking, and running a periodic staleness review — against the ~/Projects/organize repo. Use when the user says /organize, asks to capture or log a new idea, wants help filing their inbox, wants Projects scored, wants to see their current ranked list of Projects, or wants to run their periodic PARA review.
---

# Organize

## Overview

Automates the user's personal PARA/GTD system: capture, file, score, rank, and
review. Capture is a single automatic step (rewrite and save); rank is
read-only; file, score, and review each propose an action and wait for the
user to confirm before touching a file. The skill always operates on
`~/Projects/organize`, the user's real PARA repo.

## When to Use

- The user runs `/organize` or `/organize <mode>` (`capture`, `file`, `score`, `rank`, `review`).
- The user asks to jot down, capture, or log a new idea without saying where it belongs.
- The user wants help sorting `INBOX.md` into Projects, Areas, Resources, or Archive.
- The user wants Projects scored or re-scored on Impact/Effort.
- The user wants to see their current Projects ranked by Impact/Effort without re-scoring anything.
- The user wants to run their periodic review (flag stale Projects, propose archiving).
- NOT for editing arbitrary markdown files unrelated to `~/Projects/organize`.
- NOT for deciding PARA taxonomy or scoring theory from scratch — this skill applies the definitions in `~/Projects/organize`'s own `GLOSSARY.md` if present, or the ones below.

## Setup

The target repo is fixed: `~/Projects/organize`. Do not search for or ask about
this path — use it directly. If it doesn't exist, create the skeleton before
doing anything else:

```
~/Projects/organize/
  INBOX.md
  REVIEW.md
  1-projects/
  2-areas/
  3-resources/
  4-archive/
```

## Core definitions

- **Project**: a specific outcome with a finish line — work stops once reached.
- **Area**: an ongoing standard maintained indefinitely, no finish line.
- **Resource**: reference material kept for future use, not actionable itself.
- **Archive**: inactive — finished, dropped, or superseded.
- **Impact** (1-5): how much a Project moves the needle if completed.
- **Effort** (1-5): calendar time and complexity to complete a Project.
- **Quick Win**: high Impact, low Effort — do these first.
- **Thankless Task**: low Impact, high Effort — reconsider whether it belongs in Projects at all.

## File conventions

Project files use this exact shape (match it when writing or editing):

```markdown
# <Title>

<One or two sentence description of the outcome.>

Impact: X  Effort: Y

## Progress
- YYYY-MM-DD: <what happened>
```

`Impact: X  Effort: Y` is only present once a score has been proposed and
confirmed. `## Progress` entries are added by the user's own work, not
invented by this skill.

## Core Process

Determine the mode from the user's request (`capture`, `file`, `score`, `rank`,
`review`). If the user ran bare `/organize` with no mode and no clear intent,
ask which mode they want.

### Mode: capture

1. Rewrite the idea using the plain-language principles in
   `docs/plain-language-iso24495.md`: short sentences, active voice, common
   words over jargon, most-important-thing-first. Keep every fact, scenario,
   and constraint the user stated — tighten the wording, don't cut content or
   change meaning. Do not ask what domain it's in, what PARA bucket it
   belongs to, or add a score. Capture must stay a single step: rewrite and
   save without asking the user to approve the wording first.
2. Append the rewritten idea to `~/Projects/organize/INBOX.md` as a new
   bullet (or heading, if the existing file uses headings — match what's
   already there). Add nothing else.
3. Confirm briefly that it was captured, showing the rewritten wording so the
   user can see what was saved. Do not propose filing it now, even if the
   destination seems obvious — filing is a separate mode.

### Mode: file

1. Read `~/Projects/organize/INBOX.md`. If it's empty, say so and stop.
2. For each inbox item, in order:
   a. Propose a PARA bucket (Project / Area / Resource / Archive) using the
      definitions above, with one sentence of reasoning (e.g. "has a finish
      line → Project").
   b. Wait for the user to confirm or correct the bucket. Do not move on to
      the next item until this one is resolved.
   c. Once confirmed, create a new file for it under the matching folder
      (`1-projects/`, `2-areas/`, `3-resources/`, `4-archive/`), using a
      short kebab-case filename derived from the idea's title. Use the file
      convention above (Projects get the `Impact: X  Effort: Y` line only
      after Mode: score runs — leave it out for now).
   d. Remove the item from `INBOX.md`.
3. Never file more than one item without the user's go-ahead on that specific
   item — batch confirmation is not allowed, even if all buckets seem obvious.

### Mode: score

1. List every file in `1-projects/` that has no `Impact: X  Effort: Y` line,
   plus any the user explicitly names for re-scoring.
2. For each, propose an Impact (1-5) and Effort (1-5) with one sentence of
   reasoning each. Wait for confirmation or correction before writing
   anything.
3. Once confirmed, add or update the `Impact: X  Effort: Y` line in that
   Project's file.
4. After all requested Projects are scored, run Mode: rank to print the
   updated ranked list.

### Mode: rank

1. Read every file in `1-projects/` that has an `Impact: X  Effort: Y` line.
   If none do, say so and suggest running `/organize score` first.
2. Print every scored Project as a table or list of Impact/Effort, ordered
   with Quick Wins (high Impact, low Effort) first.
3. Call out any Thankless Task (low Impact, high Effort) with a one-line
   prompt to reconsider it.
4. This mode is read-only — it never proposes or writes anything, so it
   needs no confirmation step.

### Mode: review

1. Read every file in `1-projects/`.
2. For each, judge staleness from its `## Progress` section: no entry in the
   last review cycle (or ever) reads as stale. State the judgment and why.
3. For each Project you judge stale or that reads as finished, propose moving
   it to `4-archive/` (or to `2-areas/`/`3-resources/` if the conversation
   reveals it was misfiled). Wait for explicit confirmation per item before
   moving anything — staleness is a judgment call, archiving is an action the
   user approves.
4. Only after the user has responded to every flagged Project, update
   `REVIEW.md`: set "Last review" to today's date and "Next scheduled review"
   to two weeks out (or whatever cadence `REVIEW.md` already states).
5. Do not schedule or promise an automatic reminder — this review runs only
   when the user invokes it.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The PARA bucket is obviously a Project, I'll just file it" | Obvious to the agent isn't obvious to the user's actual intent — always state the reasoning and wait for confirmation, even for clear-cut cases. |
| "I already asked about the first three inbox items, I can batch the rest" | Batching defeats the propose-then-confirm model — each item gets its own confirmation, no matter how repetitive the pattern looks. |
| "The Project clearly hasn't been touched, I'll archive it" | Staleness is an observation; archiving is an action. Always propose the move and wait, even when the evidence looks conclusive. |
| "Capture is basically filing, I can suggest a bucket while I'm at it" | Capture must stay zero-decision. Suggesting a bucket at capture time reintroduces the friction the lesson eliminated. |
| "The rewrite drops a phrase, but it reads cleaner" | Plain-language rewriting is style-only — it may not drop facts, scenarios, or constraints from the original idea. If tightening the wording would lose content, keep the wording instead. |
| "I should show the rewrite and ask before saving it" | Capture stays a single step — the rewrite happens automatically, is saved, and is shown to the user afterward for visibility, not beforehand for approval. |

## Red Flags

- A file gets created, moved, or edited in `~/Projects/organize` before the user has confirmed that specific action.
- A capture turns into an unsolicited filing suggestion.
- Multiple inbox items get filed off a single confirmation.
- Scores get written without the user seeing the proposed Impact/Effort first.
- `REVIEW.md` gets updated before every flagged Project has been resolved with the user.

## Verification

- [ ] The action taken (capture/file/score/archive) matches an explicit user confirmation, not an inferred one
- [ ] Any new or edited file under `1-projects/`, `2-areas/`, `3-resources/`, `4-archive/` follows the file convention above
- [ ] `INBOX.md` only loses an item once it has a destination file
- [ ] `REVIEW.md` dates are only updated after all flagged Projects were addressed
