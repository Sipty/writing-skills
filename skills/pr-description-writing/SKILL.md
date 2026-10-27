---
name: pr-description-writing
description: "Sole authority for PR description style - use on EVERY PR body create, draft, rewrite, or feedback revision; parents delegate to pr-writer"
---

# PR Description Writing

This skill owns DESCRIPTION style only. Mechanics (title, ticket linking, template) stay with the repo's own PR-creation skill, if it has one; never edit that skill. This is the only place the framework lives.

## Delegation

- Any parent agent that creates, drafts, rewrites, or revises a PR description from feedback MUST delegate to the `pr-writer` agent and use its output. Never write the body yourself.
- If you ARE `pr-writer`, apply this skill directly; never spawn or recurse.
- Drafts go to `~/.omp/agent/pr-writer/drafts/<unique>.md`. Return the bodyfile path plus gaps (missing evidence, unknown ticket).
- Writing a draft or receiving feedback is NOT permission to publish. Editing or creating a PR body on GitHub needs per-action user approval, handled by the parent.

## Inputs to gather (writer)

1. Live diff of the PR/branch and the current PR body (if any).
2. Repo PR template (`.github/pull_request_template.md` or equivalent) for the `# Checklist`.
3. User-provided test evidence. Read ticket context only if needed for Why.
4. Tests added is not tests passed. If evidence is missing, say so and mark the check as pending; never invent results.

## Framework

```markdown
# Description

## Why
<max 2 sentences: effect/problem, then reason/constraint>

## What changed
- <max 4 concrete bullets, ordered by importance>

<optional: one relevant scope-boundary line>

# Testing
- <max 2 bullets; separate observed evidence from pending checks>

## Ticket(s)
- <links, where applicable>

# Checklist:
<repo checklist copied verbatim; checks reflect evidence, never inferred>
```

Rules:
- Describe the final diff, not the sequence of commits, iterations or superseded approaches. No file inventory, dumps, file/line references, SHAs, logs, internal/local paths, or unobserved results.
- Precise technical terms are fine when they clarify the cause; otherwise plain language, symptom before mechanism.
- Omit `Bonus` by default. Supporting changes go into the main bullets; an unrelated extra must be justified, not hidden.
- API examples only when the repo/user convention needs them: a compact fenced request showing the changed contract, never invented content.
- Length: at most ~1500-2000 chars including the checklist. Shorter is fine. No filler, risk write-ups, or AI-pipeline artifacts.

## Feedback revision

1. Reread this skill, the current draft, and the exact feedback.
2. Revise the current body, not a fresh one.
3. Classify the feedback: fact correction or one-PR exception, vs lasting preference.
4. For a lasting preference, derive the minimal reusable wording/structure/evidence/scope constraint, with no PR-specific nouns, and update the matching existing rule in this skill (managed skill only) instead of appending an overlapping one. Keep it precise and checkable, compatible with useful reviewer background.
5. An explicit user preference overrides the current framework. If scope is materially ambiguous, resolve it briefly; do not guess broad restrictions.
6. Return the bodyfile and a one-line generalized rule diff (or "no durable rule change").

## Gotchas

- The validation pipeline can rewrite PR bodies on every push; a hand-written body only sticks if re-applied after the run goes green.
- PRs open as draft (create, then `gh pr ready <n> --undo` if the tooling cannot open drafts directly).
