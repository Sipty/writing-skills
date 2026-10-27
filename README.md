# writing-skills

Agent skills that make coding agents write PR descriptions, review replies, code reviews and code comments the way a careful senior engineer does: short, specific, and not obviously written by an AI.

They are plain [Agent Skills](https://agentskills.io) (`SKILL.md` folders), so they work in Claude Code, [omp](https://github.com/can1357/oh-my-pi), and other agents that load skills.
The GitHub steps use the [`gh` CLI](https://cli.github.com).

| Skill | Use it when | What you get |
|---|---|---|
| [`pr-description-writing`](skills/pr-description-writing/SKILL.md) | Opening or revising a PR | Why / What changed / Testing / Checklist, under ~2000 characters, no invented test results |
| [`pr-thread-replies`](skills/pr-thread-replies/SKILL.md) | Answering review comments on your PR | `Done`-style one-line replies on an editable HTML page; posts only what you approve, then resolves threads |
| [`pr-review-lens`](skills/pr-review-lens/SKILL.md) | Reviewing someone else's PR | Findings by lens (silent coercion, test weight, partial failure, ...) with severity tags and one-sentence comments; drafts only |
| [`code-comment-style`](skills/code-comment-style/SKILL.md) | Writing or trimming code comments | 1-3 line comments that state the why, with no ticket IDs or session leftovers |

Optional: [`agents/pr-writer.md`](agents/pr-writer.md) is a subagent that owns all PR description drafts, for agents that support subagents. The install block below includes it.

## Install

Clone the repo, then run the lines for your agent:

```bash
git clone https://github.com/Sipty/writing-skills.git

# Claude Code
mkdir -p ~/.claude/skills ~/.claude/agents
cp -R writing-skills/skills/* ~/.claude/skills/
cp writing-skills/agents/pr-writer.md ~/.claude/agents/      # optional subagent

# omp
mkdir -p ~/.omp/agent/skills ~/.omp/agent/agents
cp -R writing-skills/skills/* ~/.omp/agent/skills/
cp writing-skills/agents/pr-writer.md ~/.omp/agent/agents/   # optional subagent
```

Or copy only the ones you want; each skill is self-contained.

To update later, `git pull` and copy again. Copying overwrites your installed files, so back up any skill you have calibrated (see below) or diff it against the new version first.

## Make them yours

The rules came from one engineer's real edits, so the examples are in their voice.
They get noticeably better once they learn yours:

- `pr-thread-replies` has a calibration table: after a few rounds, replace the examples with drafts you actually rewrote.
- `pr-review-lens` quotes real review comments: swap in ten of your own.
- `pr-description-writing` turns lasting feedback into rule edits; keep your copy writable so it can.

## Safety

None of the skills post on their own.
Replies, reviews and PR bodies are drafts until you approve each one, and review comments never quote customer identifiers.

## License

MIT, see [LICENSE](LICENSE).
