---
name: locate-feature
description: Dispatches a read-only subagent to locate the smallest relevant part of an unfamiliar, potentially large and multi-technology codebase and trace how it works, then reports Likely owner / Functionality path / Evidence / Unknowns / Next best action. Works on any tech stack — nothing about the stack is hardcoded. Use when the user explicitly types /locate-feature <question>; do not use just because a plain-language "where is X" question appears in normal conversation.
---

# Locate Feature

## Overview

Finds where an unfamiliar piece of functionality lives in a codebase, given a business question, bug report, feature name, message name, UI behavior, domain term, or error. Dispatches a subagent to search first and reason second, then returns a short, structured report instead of a wall of exploration.

## When to Use

- Invoked explicitly as `/locate-feature <question>` — e.g. "What handles Alternative Proof?", "Where is IEAT451A processed?", "What causes this declaration state change?".
- Works on any codebase or tech stack (Java/Spring, BPMN, Kafka, Drools, REST/SOAP, databases, or anything else) — no stack assumptions are baked in.
- NOT auto-triggered. Do not reach for this skill just because a plain-language "where is X handled?" question appears in conversation — only run it when the user types `/locate-feature` (or explicitly asks to run it).
- NOT for making changes. This skill only investigates and reports; it never edits code.

## Core Process

### 1. Get the question

If the user gave a question as an argument, use it as-is. If `/locate-feature` was invoked with no question, ask the user what they want located before doing anything else — do not dispatch a subagent on an empty query.

If the question is a follow-up to a prior `/locate-feature` run, ask the user to include the prior "Known" findings in the question text (or pull them from the visible conversation history if already present). Each run is a fresh, independent dispatch — there is no session memory between invocations.

### 2. Dispatch a subagent

Spawn a subagent (a fresh, generic one — e.g. `general-purpose` in Claude Code) and hand it the full briefing below as its prompt, with `<QUESTION>` replaced by the actual question:

```
Investigate this question about an unfamiliar codebase: <QUESTION>

Investigate, do not modify. Do not edit or write any files — read and search only.

Strategy:
1. Extract search anchors from the question: domain/business terms, message
   or error names, enum/constants, API endpoints, task/event names, database
   fields, class or method names — or whatever the equivalent is for this
   codebase's actual stack.
2. Search broadly with cheap tools (grep/glob/symbol search) before reading
   full files.
3. Rank candidate modules/services/components by relevance.
4. Narrow progressively — drop low-relevance candidates as evidence comes in.
5. Trace relationships only when evidence connects them, for example:
   entry point -> handler/controller/consumer -> service/domain logic ->
   workflow/rule/integration -> persistence or outgoing action. The real
   path varies by feature and stack; this is illustrative, not fixed.
6. Follow cross-service/cross-module connections only when evidence supports
   it: API calls, event/queue messages, imports/dependencies, workflow
   definitions, shared libraries, database usage, message producers/consumers.
7. Prefer symbol search and small code excerpts over reading whole files.
8. Keep a compact Known / Unknown / Next-action state as you go, so you never
   re-search something you've already found.

Constraints:
- Do not assume any particular tech stack (e.g. Java/Spring) going in — discover
  it from the repo itself, unless the question already names one.
- Optimize for low token usage. Search first, reason second.
- Do not explore the whole repo architecture unless the question requires it.
- Do not read services/modules unrelated to the question.
- Do not re-inspect a file you've already covered.
- Say plainly which facts are confirmed and which are your best guess.
- If more than one location looks plausible, rank them by confidence.
- Stop as soon as you have enough evidence to point a developer to the right
  place — you do not need to fully explain the whole feature.

Report back in exactly this format:

### Likely owner
The most relevant service/module/component.

### Functionality path
A short execution/data-flow trace.

### Evidence
Concrete references only — file paths, class/method names, workflow
definitions, message names, rules, or endpoints you actually found.

### Unknowns
What you could not confirm.

### Next best action
The smallest next search or file read that would most reduce uncertainty.
```

### 3. Relay the report

Return the subagent's report to the user as-is, in the same five-section format. Do not add your own extra investigation on top of it — if the user wants to go further, they run `/locate-feature` again with a follow-up question that includes the current "Known" findings.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I already sort of know this codebase, I'll just answer directly" | The point of dispatching a subagent is to keep the exploration's search noise out of the main conversation and to force search-before-guessing discipline. Dispatch it anyway. |
| "The question is vague, I'll guess what they mean and search" | A vague question wastes the subagent's search budget. Ask a brief clarifying question first if the anchors aren't clear, or let the subagent extract its own anchors — don't invent scope. |
| "This looks like a Java/Spring shop, I'll tell the subagent that upfront" | Don't hardcode stack assumptions into the briefing. Let the subagent discover the actual stack; feed stack hints only if the user's question already states them. |
| "The subagent found the file, I might as well fix the bug while I'm here" | This skill is investigate-only. Report the location; let the user (or a separate task) decide whether and how to change code. |

## Red Flags

- Running this skill without an explicit `/locate-feature` invocation.
- Skipping the subagent dispatch and answering from the main conversation's own guesses.
- A report missing one of the five required sections.
- The subagent editing or writing files.
- Re-running a full investigation from scratch on a follow-up instead of asking the user to carry forward the prior "Known" state.

## Verification

- [ ] The question was either given explicitly or asked for before any dispatch
- [ ] A subagent was actually dispatched with the full briefing (not answered from memory)
- [ ] The subagent's report has all five sections: Likely owner, Functionality path, Evidence, Unknowns, Next best action
- [ ] Evidence entries are concrete (real file paths / symbol names), not vague descriptions
- [ ] No files were edited or written during the investigation
