---
name: pr-review-lens
description: "Review someone else's PR like a senior engineer: what to flag, what to let pass, and one-sentence comments in a human voice. Drafts only, never posts. Use when reviewing a PR number/URL, branch or diff, or as one reviewer in a multi-reviewer panel."
---

# PR review lens

A review lens distilled from a real engineer's review history (42 PRs, 40 inline comments, 44 reviews) plus their written engineering rules.
It decides *what to look for* and *how to say it*.
It never posts: the output is a draft the user picks from.

## 1. Gather

1. `gh pr view <pr> --json number,title,body,author,baseRefName,headRefName,files` and `gh pr diff <pr>`.
2. Read the linked ticket if the body names one, and any notes or open questions the author left on their own diff.
3. Read the repo's `AGENTS.md` / `CONTRIBUTING.md`; its conventions outrank this lens on style, never on correctness.
4. Follow each change outward into the surrounding source: callers that were not updated, the sibling method that does it differently, the test that no longer covers the branch it claims to.
   Use a worktree or `gh api repos/<owner>/<repo>/contents/<path>?ref=<head>`; the diff alone hides most of what matters.

Done when every changed file has been read in context, not just in the hunk.

## 2. Apply every lens

Check each lens on every PR. Report only hits; a lens with nothing to say stays silent.

### Silent coercion
Missing or bad data must stay visibly missing. Flag anything that turns null, NaN, undefined, expired or zero into a valid-looking value, row, or response.
> "`Number(null) === 0`. No throw, no NaN, just `0.0` on the wire."
> "a zero or expired estimate silently becomes written-and-published synthetic rows instead of a 422"

### Boundary validation
Validate at the edge, whitelist over string building, unmapped values fail loudly. Check every input the PR newly accepts (dates, placeholders, enum-ish strings, ID lists in queries).
> "a whitelist lookup instead of concatenation, where an unmapped value yields `undefined`"
> "I think you should also check for valid begin, otherwise the API could accept future-starting products"

### Partial failure
Streams, batches, chunked writes, retries: what happens when step 1 succeeds and step 2 fails? Ask it as a question.
> "Will this handle mid-stream failures, ie if first chunk delivers, but second fails?"

### Test weight
Usually the most frequent finding: fewer, sharper tests. Keep contract tests, injection and validation guards, real regressions, and the branch that is actually wired in.
Flag as removable: tests pinning SQL or query substrings or exact wording/layout, pass-through wiring tests, mock echoes, near-duplicates that should be one parametrized test, tests for code scheduled for deletion or throwaway tools.
Also flag the opposite gap: only the legacy or flag-off branch is covered.
Go test file by test file: each bloated file gets its own finding naming what to keep, so the author gets a keep-list, not a vague "too many tests".
> "Most of these pin SQL substrings, which breaks on any refactor; could we keep only the injection/UUID guard, ID batching and the ifNull regression"
> "Could this shrink to default, flag-beats-env, unknown env, option conflicts and the round-trip?"
> "The test also only covers the flag-off (legacy) branch"

If the repo's conventions ask for comprehensive tests, frame test-weight findings as suggestions, not blockers.

### Dead surface and layering
Unused params, duplicate endpoints or listings, new config knobs nobody asked for, shared helpers importing from a feature package, shims or aliases left behind after a cutover.
> "The service ignores view, I would just drop it."
> "This makes `_shared` depend on `reports`, which inverts the layering"

### Root cause
A fix that cleans up the symptom (dedupe after the fact, filter on read) while the write path keeps producing bad data. Point at where it should be stopped.

### Rollout and parity
Anything that flips a customer, backend, or feature flag: is there observed parity evidence, a live-diff path, a way back? Forced-on states that never turn off belong in a runbook.
> "no live-diff path, so this one also switches backends unobserved when we flip it on."

### PR shape
Two independent changes glued together (e.g. a CI change riding on a refactor): name the split point.
Comments and docs that describe code this PR deleted.
> "I read this as two changes glued together"
> "a couple of comments that now describe a world this PR deleted"

### Comments in the diff
Code comments should be one to three lines stating the why or the invariant, readable with only the repo. Flag ticket IDs, reviewer names, dates or plan numbers in comments or test names, and multi-paragraph comment blocks.

### UI changes
If the PR touches a UI and the ticket has screenshots or mockups, compare the running result against them, pixel by pixel. Say so if the check could not be done because there is no local setup.

## 3. Calibrate

Let these pass without comment: tested dependency bumps, flag-removal cleanups, translations, mechanical refactors, small tested features in familiar code.
If that is all the PR is, the review is a one-line approve; do not hunt for findings.

Precision bar: a `blocker` or `major` names the concrete input or sequence that triggers it, verified against the head source, not inferred from the hunk.
A risk you cannot pin to a trigger is a question for "say in person", not a finding.
When the author asked questions on their own diff, answer those first; agreeing is a valid reply ("Honestly I'd do it the same way").

Severity tags:
- `blocker`: will bite in production.
- `major`: will bite under a plausible input.
- `minor`: concrete defect that will not bite on its own.
- `nit`: naming, a removable test, a stale comment. Nits are welcome; label them.

Customer identifiers (user IDs, account IDs, emails, addresses) never appear in a comment. Name the line and the shape of the problem.

## 4. Voice

Write every draft comment the way a busy colleague types:
- Exactly one sentence per finding, however long it needs to be. A second sentence means it is two findings, or the extra belongs in "say in person".
- Casual, lowercase-friendly, no headings, no bullet lists inside a comment.
- Questions for edge cases and intent ("how confident are we that `item` is only None and not a string?").
- "I'd drop it", "could we keep only X", "leaving it to you to make the call" when the fix is cheap or optional.
- A concrete code suggestion when the fix is safety-relevant (`raw == null ? null : Number(raw)`).
- `nit:` prefix for nits; humour is fine when it is natural.
- Inline comments carry no praise, SHAs, or proof-of-work; warmth goes in the summary ("fun bug", thanks for good notes).
- Keep the register low even when the tag is `major`: "Very small nitpick below, otherwise good to go".

Open questions that read better in person go in a separate "say in person" list, not into the comments.

To make the voice yours, swap the quoted examples above for real comments you have written; ten good ones beat any style description.

## 5. Output

Present in chat, one message:

```
## <title> (#<n>) by <author>

Summary (review body draft): one or two short sentences, first person: what stood out, then the verdict. Example: "Left a few comments around test reduction; the logic otherwise looks good to me".

| # | Tag | Where | Lens | Draft comment |
|---|-----|-------|------|---------------|
| 1 | major | path.ts:42 | silent coercion | <one sentence> |

Say in person: <optional, 0-2 lines>
```

Verdict line:
- Nothing worth saying: "LGTM" (or an empty approve).
- Findings of any severity: approve conditionally, body `LGTM with the comments addressed`.
- Several stacked risks that make the PR wrong as shaped: "I'd request changes", stated as advice; the user picks the event.

Then stop and ask which numbers to post, as-is or edited.
Posting is a separate step that needs the user's go-ahead.
Safest route: a pending review only the user can see, which they submit themselves. Create it with `gh api repos/<owner>/<repo>/pulls/<n>/reviews --input <file>`, where the file holds `{"commit_id": "<head sha>", "body": "...", "comments": [{"path": "...", "line": 42, "side": "RIGHT", "body": "..."}]}` and no `event` field.
