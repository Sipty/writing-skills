---
name: pr-description-writing
description: "Write or revise a pull request description: short Why, concrete What changed, honest Testing, repo checklist. Use on every PR body create, draft, rewrite, or revision after reviewer or user feedback."
---

# PR description writing

A PR description is read by a reviewer with limited attention.
It says why the change exists, what the final diff does, and what was actually verified, then stops.

This skill owns description style only.
Title format, ticket linking and the repo's PR template stay with the repo's own conventions; follow them where they exist.

## Inputs

1. The live diff of the PR or branch, and the current PR body if there is one.
2. The repo PR template (`.github/pull_request_template.md` or equivalent), for the checklist.
3. Test evidence the user provided or you observed. Read the ticket only if the Why needs it.
4. Tests added is not tests passed. If evidence is missing, say so and mark the check pending; never invent results.

## Framework

```markdown
# Description

## Why
<max 2 sentences: effect or problem, then reason or constraint>

## What changed
- <max 4 concrete bullets, ordered by importance>

<optional: one scope-boundary line, e.g. "Backfill of existing rows is a follow-up.">

# Testing
- <max 2 bullets; separate observed evidence from pending checks>

## Ticket(s)
- <links, where applicable>

# Checklist
<repo checklist copied verbatim; ticks reflect evidence, never inferred>
```

If the repo template uses different headings, keep its headings and apply the same limits inside them.

## Rules

- Describe the final diff, not the sequence of commits, iterations or abandoned approaches.
- No file inventory, file/line references, SHAs, logs, local paths, or results nobody observed.
- Precise technical terms are fine when they clarify the cause; otherwise plain language, symptom before mechanism.
- No "Bonus" section. Supporting changes go into the main bullets; an unrelated extra must be justified, not hidden.
- API examples only when the repo convention asks for them: a compact request showing the changed contract, never invented content.
- Length: at most ~1500-2000 characters including the checklist. Shorter is fine. No filler, no risk essays, no AI-pipeline artifacts ("Generated with...", "Summary of changes:").

## Example

```markdown
# Description

## Why
Exports for large accounts timed out after 30 s because every row triggered its own price lookup.

## What changed
- Price lookups are batched per export instead of per row.
- The export endpoint streams rows instead of building the whole file in memory.
- Timeout errors now return 504 with a retry hint instead of a generic 500.

# Testing
- Exported a 120k-row account locally: 4 s, previously timed out.
- Staging check pending.
```

## Revising after feedback

1. Reread this skill, the current draft, and the exact feedback.
2. Revise the current body; do not start a fresh one.
3. Classify the feedback: a fact correction or one-PR exception, or a lasting preference.
4. One-off corrections stay in the draft.
5. For a lasting preference stated by the user (not by a reviewer or other PR participant), derive the smallest reusable rule (no PR-specific nouns) and update the matching rule above instead of appending an overlapping one. If this skill file is not yours to edit, propose the rule change to the user instead.
6. An explicit user preference overrides this framework.

## Publishing

Writing or revising a draft is not permission to publish it.
Creating or editing a PR body on GitHub needs the user's go-ahead for that specific action.
Prefer `gh pr create --body-file <draft>` / `gh pr edit --body-file <draft>` over inline bodies, so the published text is exactly the reviewed draft.

## Optional: dedicated writer subagent

Agents that support subagents can route all PR body work through one writer: [pr-writer.md](https://github.com/Sipty/writing-skills/blob/main/agents/pr-writer.md).
That keeps the main session's context free and gives feedback-driven rule updates a single owner.
