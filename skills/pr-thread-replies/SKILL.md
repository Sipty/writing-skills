---
name: pr-thread-replies
description: "Draft replies to GitHub PR review threads in the user's own short voice, hand them over on an editable HTML page, then post exactly what the user approved and resolve threads. Use when answering review comments on the user's PR."
---

# PR thread replies

Replies go out under the user's name, so the user edits and approves every one.
Flow: collect threads -> draft in the user's voice -> editable HTML page -> user exports JSON -> post exactly that -> resolve -> learn from the user's edits.

Needs: `gh` authenticated as the user, and the `template.html` next to this file.

## 1. Collect threads

- Inline comments and their ids: `gh api repos/<owner>/<repo>/pulls/<n>/comments --paginate`. The `id` is the `r<id>` in discussion URLs.
- Thread node ids for resolving, via GraphQL: `reviewThreads(first: 50) { nodes { id isResolved comments(first: 1) { nodes { databaseId } } } }`; match on the first comment's `databaseId`.
- One card per unresolved inline thread. Add one optional "General PR comment" card (`send: false`) summarising top-level review bodies the user might answer.
- Questions addressed to someone else (reviewer A asking reviewer B) are not the user's to answer: mention them on the general card only.

## 2. Draft in a human voice

Reviewers can tell when a reply was written by an AI. Rules, in priority order:

1. Suggestion applied as proposed -> `Done`. Never restate the reviewer's own idea back to them.
2. Same fix as another thread -> `Done, see above.`
3. Add a clause only when the reviewer cannot infer what happened: the fix differs from the suggestion, something was removed instead of changed, or a choice was made. One line: `Removed, provisioning alone should be enough now.`
4. Hedge outcomes not proven in the reviewer's environment: "should be enough", not "is enough".
5. No implementation tour (class names, mechanics, test counts); that belongs in commits or the PR description.
6. Never: gratitude, "great catch", apologies, "I've updated", file:line evidence, SHAs, bullet lists, headings, multiple paragraphs, em dashes.
7. Disagreeing or deferring: one plain sentence with the reason, casual and downplayed. Push real product decisions back to the user instead of drafting an answer.
8. Optional or musing comments (side info, future ideas, "an alternative would have been X") default to `send: false`; the user can resolve those silently.
9. Partially addressed design comments (fix done, but a decision stays with the reviewer): reply with `resolve: false`, so the reviewer closes it.

## 3. Build the handover page

1. Copy `template.html` and replace two tokens with JSON:
   - `__CONFIG_JSON__` -> `{"repo": "<owner>/<repo>", "pr": <n>}`
   - `__DRAFTS_JSON__` -> array of `{key, commentId, author, location, comment, change, reply, send, resolve}` (`commentId: null` for the general card).
2. Generate it with a JSON serializer rather than by hand, e.g. `html.replace("__DRAFTS_JSON__", json.dumps(drafts))` in Python.
3. Save it outside git (`/tmp/pr-<n>-replies.html`), open it (`open` on macOS, `xdg-open` on Linux), and tell the user: edit, untick what should not go out, then Download or Copy the JSON.

The page stores edits in the browser's localStorage and sends nothing itself.

## 4. Post

- The exported JSON is the approval for exactly the items with `send: true`. Post `reply` verbatim: no rewording, no additions.
- If the user says "go" but no JSON arrived, ask once: post drafts as-is, post ticked only, resolve without replies, or wait for the export. Never assume, and never act on unrelated clipboard content.
- Replies like `Done` claim the fix is visible. If the fixing commits are not pushed yet, ask whether to push first.
- Reply: `gh api repos/<owner>/<repo>/pulls/<n>/comments/<commentId>/replies -f body=<reply>`.
- Resolve only when `resolve: true` (or the user said resolve all): `gh api graphql -f query='mutation($t: ID!) { resolveReviewThread(input: {threadId: $t}) { thread { isResolved } } }' -f t=<thread node id>`.
- `send: true` with `commentId: null` -> top-level comment: `gh pr comment <n> --body-file <file>`.
- Verify: re-query `reviewThreads`; every posted thread shows the reply and the expected `isResolved`.
- Re-request review: `gh api -X POST repos/<owner>/<repo>/pulls/<n>/requested_reviewers -f "reviewers[]=<login>"`.

## 5. Calibrate to your voice

After every round, diff each draft against what the user actually sent.
Patterns they always cut, hedges they always add, words they always change: fold them into the rules above or into a calibration table like this one, kept to the most instructive rows.

Example from the author's own edits:

| Draft | What was actually sent |
|---|---|
| Done, every write under `/config` now rebuilds the readouts before it responds. Kept `/reload-configs` for ... | Done |
| Removed, provisioning alone is enough now. | Removed, provisioning alone should be enough now. |
| Gone, the frontend doesn't call reload anymore. | Done |
| Done, the transport is set in config and the flag is gone. | Done. |
| Rewrote them: seed the real fixture, run the fake service, patch only external IO. | (kept) |
| Optional general comment | (not sent) |

Short one-liners plus `Done` are the register to aim for; most rewrites shorten the draft.
