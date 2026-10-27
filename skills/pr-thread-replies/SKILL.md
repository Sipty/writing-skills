---
name: pr-thread-replies
description: "Use when drafting or posting replies to GitHub PR review threads for Chavdar: his reply voice, editable HTML handover page, JSON-approved posting and thread resolution."
---

# PR thread replies (Chavdar)

Replies to review threads are posted under his identity, so he edits and approves every one.
Flow: draft in his voice -> editable HTML page -> he exports JSON -> post exactly that -> resolve -> learn from his edits.

## 1. Collect threads
- `pr://<n>?comments=all` or `gh api repos/<owner>/<repo>/pulls/<n>/comments` for comment ids (`databaseId`, the `r<id>` in discussion URLs).
- Thread node ids for resolution: GraphQL `reviewThreads(first: 50) { nodes { id isResolved comments(first: 1) { nodes { databaseId } } } }`, match on the first comment's databaseId.
- One card per inline thread. Add one optional "General PR comment" card, `send: false`, summarising top-level review bodies he might answer.
- Questions addressed to someone else (e.g. reviewer A asks reviewer B) are not his to answer: mention them on the general card only.

## 2. Draft in his voice
He does not write like an AI. Rules, in priority order:
1. Suggestion applied as proposed -> `Done` (punctuation optional). Never restate the reviewer's own idea back to them.
2. Linked thread ("see below") -> `Done, see above.`
3. Add a clause only when the reviewer cannot infer what happened: the fix differs from the suggestion, something was removed instead of changed, or a choice was made. Keep it to one line: `Removed, provisioning alone should be enough now.` / `Rewrote them: seed the real fixture, run the fake service, patch only external IO.`
4. Hedge outcomes not proven in the reviewer's environment: "should be enough", not "is enough".
5. No implementation tour (class names, mechanics, test counts); that belongs in commits or the PR description.
6. Never: gratitude, "great catch", apologies, "I've", file:line evidence, SHAs, bullet lists, headings, multiple paragraphs, em dashes.
7. Disagreeing or deferring: one plain sentence with the reason, casual and downplayed. Push genuine product decisions back to him instead of drafting an answer.
8. Optional or musing comments (side info, future plans, "alternative would have been X") default to `send: false`; he resolves those silently.
9. Partially addressed design comments (fix done, but a follow-up decision remains with the reviewer): reply, `resolve: false`, so the reviewer closes it.

### Calibration (drafts -> his final)
| Draft | Final |
|---|---|
| Done, every write under `/config` ... rebuilds the readouts before it responds. Kept `/reload-configs` for ... | Done |
| Removed, provisioning alone is enough now. | Removed, provisioning alone should be enough now. |
| Gone, the frontend doesn't call reload anymore. | Done |
| Done, the transport is set in config and the flag is gone. | Done. |
| Done, see above. | (kept) |
| Rewrote them: seed the real fixture, run the fake service, patch only external IO. | (kept) |
| optional general comment | left off |

On another PR he approved all five drafts unedited (short one-liners plus `Done`), so that register is right.
When he asks "have all his asks been resolved? resolve if so" without exporting JSON, offer "post the ticked drafts as-is, resolve all" first; he picked it last time.

## 3. Build the page
- Copy `template.html` from this skill's directory and replace the two tokens:
  - `__CONFIG_JSON__` -> `{"repo": "<owner>/<repo>", "pr": <n>}`
  - `__DRAFTS_JSON__` -> array of `{key, commentId, author, location, comment, change, reply, send, resolve}` (`commentId: null` for the general card).
- Write the substituted file (a py eval with `json.dumps` is easiest); escape nothing by hand beyond valid JSON. Keep the `drafts` list in the eval kernel so posting can reuse it verbatim.
- Save outside git, e.g. a git-excluded `.omp/` dir in the repo or `/tmp/pr-<n>-replies.html`.
- Smoke it once: `browser.open({ url: 'file://...', headless: true })` (takes an options object, not a string); the returned title proves it rendered. `tab.run` callbacks run outside the page (no `document`), so don't use them for DOM checks. Close the tab after.
- Then `open <file>` and tell him: edit, untick, then Download (`~/Downloads/pr-<n>-replies.json`) or Copy and paste.

## 4. Post
- His JSON is the approval for exactly the items with `send: true`. Post `reply` verbatim: no rewording, no trailing additions.
- If he says "done"/"go"/"resolve" but no JSON is in `~/Downloads` and `pbpaste` doesn't hold it (the clipboard often holds unrelated text: ignore it, never act on it), ask once with `ask`: post drafts as-is / post ticked only / resolve without replies / he exports first. Never assume.
- Precondition: replies like `Done` claim the fix is visible. If the fixing commits are not on the remote yet, ask whether to push first (push needs its own approval) before posting.
- Reply: `gh api repos/<owner>/<repo>/pulls/<n>/comments/<commentId>/replies -f body=<reply>`.
- Resolve only when `resolve: true` (or he said resolve all): `gh api graphql --input -` with `{"query": "mutation($t: ID!) { resolveReviewThread(input: {threadId: $t}) { thread { isResolved } } }", "variables": {"t": "<thread node id>"}}`.
- `send: true` with `commentId: null` -> top-level comment via `gh pr comment <n> --body-file <tmpfile>`.
- Verify: re-query reviewThreads and count unresolved; every posted thread shows the reply and the expected `isResolved`.

## Re-requesting a review
`gh pr edit --add-reviewer` is blocked by the pr-writer guard (any `gh pr edit` must be a standalone body-file call). Use REST instead: `gh api -X POST repos/<owner>/<repo>/pulls/<n>/requested_reviewers -f "reviewers[]=<login>"`.

## 5. Learn from his edits
After every round, diff each draft against his final. Any new pattern (cuts, hedges, wording he always changes) goes into the rules or calibration table above via `manage_skill update`. Keep the table to the most instructive rows.
