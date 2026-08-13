---
name: teach
description: Turns the current directory into a stateful teaching workspace and teaches the user a topic across multiple sessions, producing short self-contained HTML lessons tied to a mission, grounded in trusted resources, and backed by a glossary and learning records that persist between sessions. Use when the user runs /teach or explicitly asks to be taught a topic in a dedicated, ongoing way (not a single quick explanation).
disable-model-invocation: true
argument-hint: "What would you like to learn about?"
---

# Teach

## Overview

Teaches the user a topic over multiple sessions inside a dedicated teaching workspace, rather than answering with a single one-off explanation. The workspace accumulates state — the user's mission, a curated resource list, a glossary, learning records, and a growing library of lessons — so each session builds on what came before instead of starting cold. The reasoning behind these choices (why lessons stay small, why retention beats fluency) is in `references/philosophy.md` — read it when a step below needs justifying.

## When to Use

- The user runs `/teach`, or explicitly asks to be taught a topic in an ongoing, structured way ("teach me Rust", "I want to learn strength training properly", "help me get good at X over time").
- The user returns to a topic they were previously being taught and wants to continue.
- NOT for a single quick explanation of a concept the user won't return to — just answer directly.
- NOT for generating a one-off tutorial document with no persistent workspace or follow-up sessions — that's a normal write task, not this skill.

## Teaching Workspace

Treat the current directory as a teaching workspace. State lives in these files, created lazily as needed:

- `MISSION.md` — why the user is learning this. Format: `references/mission-format.md`.
- `RESOURCES.md` — trusted sources for knowledge and community. Format: `references/resources-format.md`.
- `GLOSSARY.md` — the workspace's canonical terminology. Format: `references/glossary-format.md`.
- `learning-records/0001-<slug>.md`, `0002-<slug>.md`, … — non-obvious insights that steer future sessions. Format: `references/learning-record-format.md`.
- `lessons/0001-<slug>.html`, `0002-<slug>.html`, … — the actual teaching output. See "Lessons" below.
- `assets/` — reusable components (stylesheet, quiz widgets, diagram helpers) shared across lessons.
- `NOTES.md` — a scratchpad for the user's stated preferences and working notes.

## Core Process

### 1. Establish or load the mission

If `MISSION.md` doesn't exist, interview the user before writing anything: what are they trying to achieve, and why? Push past abstract framings ("understand X") to the concrete outcome ("ship a Rust CLI to my team"). Write `MISSION.md` per `references/mission-format.md` and confirm it with the user.

If `MISSION.md` already exists, read it. It grounds every decision below.

### 2. Gather resources before teaching

Before drafting lessons, populate or check `RESOURCES.md`. Never trust parametric knowledge over a found, high-trust source — search for primary sources, recognized experts, and well-moderated communities. Follow `references/resources-format.md`. See `references/philosophy.md` for why knowledge is gathered before skills are practiced.

### 3. Find the zone of proximal development

Read `learning-records/` (if any) to see what the user already knows or has misunderstood. Pick the next thing to teach so the user is challenged just enough — not review, not over their head. If the user names an exact thing they want to learn, teach that instead of guessing.

### 4. Build the lesson

A lesson is one self-contained HTML file in `lessons/`, named `NNNN-<dash-case-name>.html` (number increments from the highest existing lesson). Each lesson:

- Teaches one tightly-scoped thing tied to the mission — small enough to complete quickly, since working memory is limited.
- Is visually clean and readable (think Tufte: strong typography, generous whitespace) — the user will return to it.
- Reuses components from `assets/` rather than inlining styles or widgets a second lesson would duplicate. If `assets/` is empty, create a shared stylesheet first — every lesson should look like part of one course, not a one-off.
- Links to one primary source from `RESOURCES.md` for the user to read or watch.
- Links via HTML anchors to related lessons and reference documents (`GLOSSARY.md`, prior lessons).
- Ends with a reminder that the user can ask the teaching agent follow-up questions.
- Teaches knowledge before skills: give only the knowledge required for the lesson's skill, then drive retention through effortful practice — retrieval questions, spacing across sessions, interleaving related-but-distinct topics (`references/philosophy.md` explains why fluency isn't the goal). For quizzes, keep every answer the same length (words and characters) so formatting doesn't leak the answer.

Open the lesson file for the user (e.g. via a CLI command) once it's written.

### 5. Capture what happened

- Promote a term to `GLOSSARY.md` only once the user has demonstrably understood it — not merely been shown it. Follow `references/glossary-format.md`.
- Write a learning record when the user shows genuine understanding of something non-trivial, discloses prior knowledge, corrects a misconception, or the mission shifts. Follow `references/learning-record-format.md`. Routine coverage does not warrant a record — wait for evidence.
- If the mission shifted, update `MISSION.md` and confirm the change with the user before treating it as settled.
- Log durable user preferences about how they want to be taught in `NOTES.md`.

### 6. Point toward wisdom when it's asked for

When a question needs real-world testing rather than more explanation, answer as best you can, then point at a high-reputation community from `RESOURCES.md` (or find one) — see `references/philosophy.md` on why wisdom needs a community, not more explaining. If the user has said they don't want to join a community, respect that and don't keep re-suggesting it — check `NOTES.md`/`RESOURCES.md` for that preference first.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The mission is obvious from what they asked, skip the interview" | A skipped mission interview produces lessons that drift from what the user actually wants, with no way to judge what to teach next. Interview first, even briefly. |
| "I'll just explain it in chat instead of writing a lesson" | Chat explanations aren't saved, aren't reusable, and don't build the glossary or resource base the next session depends on. The lesson file is the point. |
| "This term came up, I'll add it to the glossary now" | If the user hasn't shown they can use the term correctly yet, adding it records exposure, not understanding — the glossary becomes untrustworthy as a compressed record. |
| "I already know this topic, no need to search for resources" | Parametric knowledge isn't cited, isn't verifiable, and may be stale. Lessons should cite trusted, found sources, not the agent's own recall. |
| "Today's session was routine, I'll skip the learning record" | Skipping records silently degrades the zone-of-proximal-development estimate for next time — write one whenever the qualifying conditions in `references/learning-record-format.md` are met, not on a fixed schedule. |
| "I'll style this lesson inline, it's just one file" | Inline styles duplicate across lessons and drift out of sync. Shared look-and-feel belongs in `assets/`, reused every time. |

## Red Flags

- Drafting a lesson before `MISSION.md` exists or has been read.
- A lesson that covers more than one tightly-scoped idea, or would take long to work through in one sitting.
- Glossary terms added right after first exposure, with no evidence of understanding.
- Resources cited from memory with no link, or no resource cited at all.
- A learning-records directory that reads like a session diary instead of decision-grade insights.
- Repeated community suggestions after the user said they don't want one.

## Verification

- [ ] `MISSION.md` exists, was confirmed with the user, and grounds the session's teaching choice
- [ ] `RESOURCES.md` was checked or updated before teaching, and the lesson cites a primary source from it
- [ ] The lesson is a single self-contained HTML file in `lessons/`, correctly numbered, reusing `assets/` components
- [ ] The lesson is scoped to one thing, tied to the mission, and sits in the user's zone of proximal development
- [ ] Any new glossary terms reflect demonstrated understanding, not just exposure
- [ ] A learning record was written if the session met any of the qualifying conditions
- [ ] `NOTES.md` reflects any new stated preferences from the user
