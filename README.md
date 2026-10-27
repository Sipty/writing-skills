# writing-skills

Agent skills that make coding agents write PRs, review replies, reviews and code comments the way I do.
Written for [omp](https://github.com/can1357/oh-my-pi); the `SKILL.md` format also works in Claude Code and other agents that read Agent Skills.

| Skill | What it does |
|---|---|
| [`pr-description-writing`](skills/pr-description-writing/SKILL.md) | PR description framework: Why, What changed, Testing, Checklist, ~2000 chars max. Learns from feedback. |
| [`pr-writer`](agents/pr-writer.md) (agent) | Subagent that owns PR description drafts and applies the skill above. |
| [`pr-thread-replies`](skills/pr-thread-replies/SKILL.md) | Replies to review threads in a human voice ("Done"), via an editable HTML page you approve before anything posts. |
| [`chavdar-pr-reviewer`](skills/chavdar-pr-reviewer/SKILL.md) | Review lens for someone else's PR: what to flag, what to let pass, one-sentence comments. Drafts only, never posts. |
| [`code-comment-style`](skills/code-comment-style/SKILL.md) | Terse code comments: 2-3 lines max, the why or invariant, no ticket IDs. |

## Install

Copy a skill folder into your agent's skills directory, for example:

```bash
cp -R skills/code-comment-style ~/.omp/agent/skills/      # omp
cp -R skills/code-comment-style ~/.claude/skills/         # Claude Code
cp agents/pr-writer.md ~/.omp/agent/agents/               # omp subagent
```

The skills refer to "him" (me, the author) because they were written for agents working on my behalf.
Swap in your own voice examples; the calibration tables are the part worth personalising.

## License

MIT, see [LICENSE](LICENSE).
