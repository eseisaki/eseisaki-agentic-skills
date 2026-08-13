---
name: handoff
description: Writes a standalone handoff document that transfers the current task to a different agent, tool, or teammate, saved outside the repo and referencing existing artifacts instead of restating them. Use when the user explicitly requests a handoff document or asks to transfer this work to someone/something else, or runs /handoff.
disable-model-invocation: true
argument-hint: "What will the next session focus on?"
---

# Handoff

## Overview

Writes a handoff document that lets a fresh agent continue the current work without re-deriving context. The document points to durable artifacts (specs, plans, commits, PRs, issues) instead of re-explaining their contents, and is saved outside the workspace so it doesn't pollute the repo.

## When to Use

- The user explicitly asks for a handoff document, or runs `/handoff`.
- The user is about to end a session, switch agents, or hand the task to someone else and wants continuity preserved.
- NOT for a plain end-of-turn summary of what just happened — that belongs in the chat response, not a file.
- NOT for documentation meant to live in the repo long-term (specs, ADRs, READMEs) — this document is a one-time transfer artifact, not a permanent record.

## Core Process

### 1. Determine the focus

If the user passed arguments (e.g. `/handoff <focus>`), treat that text as what the next session should focus on and tailor the document around it. Otherwise, infer the focus from the current task.

### 2. Gather what's durable vs. what's only in this conversation

Check what already exists in checkable artifacts — specs, plans, ADRs, issues, commits, diffs, PRs. Anything already captured there should be referenced by path or URL, not duplicated. Only conversation-only information (decisions made verbally, dead ends tried, open questions) needs to be written out in full.

### 3. Write the document

Save it to `~/.claude/handoffs/<sanitized-project-dir>/handoff.md`, where `<sanitized-project-dir>` is the project's root path with every `/` replaced by `-` — never into the project workspace. This fixed, per-project location is what the `SessionStart` hook checks to auto-load the handoff into a fresh session; overwrite any existing file there. Include:

- **Goal** — what the overall task is trying to achieve.
- **State** — what's done, what's in progress, what's left. Reference artifacts by path/URL rather than re-describing their content.
- **Decisions** — non-obvious choices made and why, especially ones not recorded elsewhere.
- **Open questions / blockers** — anything unresolved that the next session needs to address.
- **Suggested skills** — skills the next agent should invoke to continue efficiently (name them explicitly).

### 4. Redact

Scan the draft for secrets or sensitive data — API keys, tokens, passwords, credentials, personally identifiable information — and redact them before saving.

### 5. Report the path

Tell the user the file path so they (or the next agent) can find it. Mention that a `SessionStart` hook will auto-load it into the next session opened in this project, so they don't have to point Claude at it manually. Don't print the full document contents into the chat unless asked.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll just paste the plan into the handoff doc too" | If the plan already lives in a file, commit, or issue, duplicating it drifts out of sync the moment either copy changes. Link to it instead. |
| "No need to check for secrets, this is just an internal summary" | Handoff docs get read by other agents and sometimes shared outside the original context — redact regardless of where it's headed. |
| "I'll save it in the repo so it's easy to find" | That turns a one-time transfer artifact into repo clutter. Use the fixed `~/.claude/handoffs/<sanitized-project-dir>/` location and report the path instead — it's what the `SessionStart` hook reads to auto-load it next time. |

## Red Flags

- The handoff document re-explains content that's already in a spec, plan, commit message, or PR description.
- Secrets, tokens, or credentials appear in the draft unredacted.
- The document is saved inside the project workspace, or anywhere other than `~/.claude/handoffs/<sanitized-project-dir>/handoff.md`.
- No "suggested skills" section, leaving the next agent to rediscover which skills apply.

## Verification

- [ ] Document references existing artifacts by path/URL instead of duplicating their content
- [ ] Secrets and PII are redacted
- [ ] Saved to `~/.claude/handoffs/<sanitized-project-dir>/handoff.md`, not the project workspace
- [ ] Includes a "Suggested skills" section
- [ ] User was told the file path
