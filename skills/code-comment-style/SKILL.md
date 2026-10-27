---
name: code-comment-style
description: "Write and tighten code comments so they state the why or the invariant in 1-3 lines. Use when adding comments to code, or when pruning verbose comments in a diff or PR."
---

# Code comment style

A comment is for a reader who has only the repo: no ticket, no PR thread, no chat that produced the code.
It earns its place by saying something the code cannot: why the obvious-looking version is wrong, which invariant must hold, or which constraint forced the shape.

## Rules

1. **Max 2-3 lines per comment.** If it needs more, it is explaining the wrong thing. Keep only the non-obvious invariant; cut narrative, chronology, and impact stories.
2. **One comment per concern, then "same treatment" for repeats.** Two identical loops do not each get a 6-line essay; the second gets `// Same upsert treatment for summaries.`
3. **No session artifacts.** No ticket IDs, PR numbers, reviewer names, dates, or plan step numbers in comments or test names. `describe('idempotent persist on recompute (PROJ-123)')` becomes `describe('idempotent persist on recompute')`. The PR and the ticket tracker own those references.
4. **No index or constraint-name recitals** when the column tuple says it shorter: `(account, timestamp, quality)` beats `reading_account_timestamp_quality_idx`.
5. **Explain why, not what.** "create() on an id already in the identity map resets the change-tracking snapshot, so the UPDATE is silently skipped" beats "set the fields on the entity".
6. **File and module headers** state purpose and the non-obvious decisions, not a changelog.
7. **Constants** get a short note on what they control when the name alone does not say it.

## Before and after

Before (5 lines):
```
// Flush the deletions before persisting: within a single flush the ORM runs
// INSERTs before DELETEs, so a recomputed row that lands on a slot
// (account_id, timestamp, quality) still held by a row with a different
// deterministic id would violate reading_account_timestamp_quality_idx and
// abort the whole flush, freezing the account's series (PROJ-123).
```
After (2 lines):
```
// Flush deletions first: within one flush the ORM runs INSERTs before DELETEs, so
// a replacement row would collide on (account, timestamp, quality) and abort the flush.
```

Before (5 lines, plus a near-identical twin further down):
```
// Upsert readings. For a reading that keeps its deterministic id, set the changed
// fields ... Only timestamp, value, quality and seqId are touched; id and account form
// the identity, and re-assigning the account relation would add the entity to ...
// For a summary that already exists, set the changed fields ... a second time.
```
After (3 lines + 1):
```
// Upsert readings: create() on an id already in the identity map resets the change-
// tracking snapshot, so the UPDATE would be silently skipped - set the fields on the
// loaded entity instead.
...
// Same upsert treatment for summaries.
```

## When tightening existing comments

- Never let a comment trim change code. If the comment sits next to declarations, re-read the region after editing and confirm the code lines survived.
- Re-run the affected tests after touching comments near test titles.
- If the repo has its own comment conventions (`AGENTS.md`, `CONTRIBUTING.md`, linters), they win.
