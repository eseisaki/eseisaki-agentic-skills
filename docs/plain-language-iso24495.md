# Plain Language (ISO 24495-1)

A reference for writing clearly, based on the principles in ISO 24495-1:2023
*"Plain language — Part 1: Governing principles and guidelines."* This is a
practical distillation for agents drafting docs, messages, PRs, and other
prose — not a substitute for the published standard, which is copyrighted
and should be consulted directly for certification or legal purposes.

## The definition

A piece of writing is in plain language when the intended reader can:

1. **Find** what they need,
2. **Understand** what they find, the first time they read it, and
3. **Use** that information to do what they need to do.

If any of these three fail, the text isn't plain — no matter how
grammatically correct or "professional" it sounds.

## The four principles

### 1. Relevant, correct, complete content

Include what the reader needs to act — no more, no less.

- Cut content the reader doesn't need to complete their task.
- Don't omit content the reader needs, even if it's inconvenient.
- Verify facts, numbers, and instructions before including them.

### 2. Reader, purpose, and context focus

Write for the actual reader, not for yourself or an idealized expert.

- Identify who reads this and what they're trying to do with it.
- Match vocabulary and depth to that reader's background — not the
  author's.
- Address the reader directly ("you") where natural.

### 3. Organization that helps the reader find things

Structure the document as a path, not a pile.

- Put the most important information first.
- Group related content under clear, descriptive headings.
- Use lists, tables, and steps instead of burying sequence in prose.
- One idea per paragraph; one main point per sentence.

### 4. Clear wording and design

Make individual sentences and the page itself easy to process.

- Prefer short, common words over jargon or formal register
  ("use" not "utilize", "start" not "commence").
- Prefer active voice and a clear actor ("the system deletes the file",
  not "the file is deleted").
- Keep sentences short; split compound sentences that bury the main
  point in subordinate clauses.
- Define any term the reader can't be assumed to know, on first use.
- Use whitespace, headings, and formatting to support scanning —
  design is part of clarity, not decoration.

## Practical checklist for agent output

Before sending a substantive piece of writing (a report, a doc, a long
message), check:

- [ ] Would the intended reader know what to do after reading this once?
- [ ] Is the most important thing said first, not buried at the end?
- [ ] Are there sentences over ~25 words that could split into two?
- [ ] Is there jargon, acronyms, or nominalizations ("implementation of")
      that could be plainer verbs ("implement")?
- [ ] Does structure (headings/lists) match how the reader will scan it,
      or is it a wall of prose?
- [ ] Is anything included that the reader doesn't actually need?
