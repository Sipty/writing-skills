---
name: code-comment-style
description: "Writing and tightening code comments in Chavdar's preferred style - use when adding comments to code or pruning verbose comments in a PR"
---

# Code Comment Style

Chavdar's comment style, refined when he asked to tighten the comments on a PR. Builds on his general rules (terse 1-2 lines, written for a reader with only the repo, no session artifacts).

## Rules

1. **Max 2-3 lines per comment.** If a comment needs more, it is explaining the wrong thing. Keep only the non-obvious invariant; cut narrative, chronology, and impact stories ("freezing the account's series" does not belong in code).
2. **One comment per concern, then "same treatment" for repeats.** Two identical upsert loops do not each get a 6-line essay - the second gets `// Same upsert treatment for summaries.`
3. **No ticket IDs in code or test names.** `describe('idempotent persist on recompute (PROJ-123)')` -> `describe('idempotent persist on recompute')`. PR body and ticket tracker own ticket references.
4. **No index/constraint-name recitals** when the column tuple says it shorter: `(account, timestamp, quality)` beats `reading_account_timestamp_quality_idx / the summary unique index`.
5. Prefer explaining WHY the obvious-looking code is wrong (e.g. "create() on an id already in the identity map resets the change-tracking snapshot, so the UPDATE is silently skipped") over restating WHAT the code does.

## Approved before/after

Before (5 lines):
```
// Flush the deletions before persisting: within a single flush MikroORM runs
// INSERTs before DELETEs, so a recomputed row that lands on a slot
// (account_id, timestamp, quality) still held by a row with a different
// deterministic id would violate reading_account_timestamp_quality_idx / the summary
// unique index and abort the whole flush, freezing the account's series (PROJ-123).
```
After (2 lines):
```
// Flush deletions first: within one flush MikroORM runs INSERTs before DELETEs, so
// a replacement row would collide on (account, timestamp, quality) and abort the flush.
```

Before (6 lines + a 7-line twin):
```
// Upsert readings. For a reading that keeps its deterministic id, set the changed
// fields ... Only timestamp, value, quality and seqId are touched; id and account form
// the identity, and re-assigning the account relation would add the entity to ...
// create new summaries
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

## When tightening

Never let a comment trim change code. If the comment sits inside a range with declarations, re-read the region after editing and verify the code lines survived (a comment-range edit can sweep declarations with it). Re-run the targeted tests after touching comments near test titles.
