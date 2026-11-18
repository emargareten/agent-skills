# Agent Skills

Three focused, reusable coding-agent skills for understanding, reviewing, and simplifying changes.

| Skill | Purpose | Changes code? |
| --- | --- | --- |
| [`explain-changes`](./explain-changes/SKILL.md) | Explain what changed and why, grouped by purpose | No |
| [`review`](./review/SKILL.md) | Find and verify actionable defects, regressions, and spec mismatches | Only with `--fix` |
| [`simplify`](./simplify/SKILL.md) | Apply small, safe, behavior-preserving cleanups | Yes |

## Install

Each directory contains a standalone `SKILL.md` with YAML frontmatter. Copy the desired directories into your agent's skills directory, for example:

```bash
# From the cloned repository:
mkdir -p ~/.agents/skills
cp -R review simplify explain-changes ~/.agents/skills/
```

Agent products may use different skill locations or invocation conventions; adjust the destination to your agent's documentation.

## Usage

Invoke the relevant skill by name in a compatible coding agent:

- `explain-changes`: summarize the current diff or a specified PR/branch/commit/path.
- `review`: review the current diff or a specified PR/branch/path without edits; add `--fix` to request targeted fixes after review.
- `simplify`: clean up the current diff or a specified PR/branch/commit/range/path and run relevant checks.

With no target, each skill compares against the PR base or default-branch merge base (then upstream, then `HEAD~1`) and includes uncommitted and relevant untracked changes.

These skills work independently. `review` deliberately overlaps with the cleanup angles of `simplify`, but only the latter automatically applies improvements.

`review` reports findings through a `ReportFindings` tool when the agent provides one, and otherwise falls back to a plain severity-ordered list.

## Design principles

- Review changes in context, not only isolated diff hunks.
- Prefer concrete scenarios and evidence over generic warnings.
- Respect repository instructions and existing conventions.
- Preserve unrelated work and avoid unnecessary refactors.
- Report uncertainty and checks actually performed.

## License

MIT. See [LICENSE](./LICENSE).
