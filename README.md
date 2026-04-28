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

- `explain-changes`: summarize the current diff or a specified PR/branch/path.
- `review`: review a diff without edits; add `--fix` to request targeted fixes after review.
- `simplify`: clean up the scoped changes and run relevant checks.

These skills work independently. `review` deliberately overlaps with the cleanup angles of `simplify`, but only the latter automatically applies improvements.

## Design principles

- Review changes in context, not only isolated diff hunks.
- Prefer concrete scenarios and evidence over generic warnings.
- Respect repository instructions and existing conventions.
- Preserve unrelated work and avoid unnecessary refactors.
- Report uncertainty and checks actually performed.

## License

MIT. See [LICENSE](./LICENSE).
