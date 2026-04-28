---
name: simplify
description: Review changed code for reuse, simplification, efficiency, and root-cause-level design; automatically apply safe, behavior-preserving improvements and verify them. Use when asked to simplify, clean up, or refactor a diff without performing a correctness review.
---

# Simplify changed code

Improve the **quality of changed code**, not its correctness. Identify unnecessary duplication, complexity, wasted work, and fixes applied at the wrong abstraction level. Apply focused, behavior-preserving improvements. **Do not hunt for bugs**; leave correctness and security reviews to their dedicated workflows.

## Input and scope

Accept an optional PR, branch, commit/range, or file/directory argument. If supplied, use that target. Otherwise:

1. Inspect `git status --short` and identify the current branch and upstream.
2. Prefer the merge-base diff against the actual PR target branch when available. Otherwise use the repository default branch; use a suitable upstream tracking branch as fallback, then `HEAD~1` if necessary. Do not assume upstream is the PR base.
3. Include uncommitted tracked changes using `git diff HEAD`, and inspect untracked files relevant to the task. Avoid counting the same changes twice.
4. Use the resulting changes to define review scope. Read surrounding code, tests, and existing helpers for context, but do not expand the refactor unnecessarily.

If the target is ambiguous or the repository has no usable baseline, state the chosen scope before proceeding. Never discard or overwrite unrelated user changes.

## Core decision rules

**Deletion first; abstractions last.** Prefer removing unnecessary code, state, branches, and special cases, then reusing existing capabilities, before adding a new helper, service, layer, or dependency. Optimize for fewer concepts and easier maintenance, not minimum line count.

**Doing nothing is valid.** Do not manufacture changes. A little local duplication may be clearer than an abstraction; straightforward code may be better than a clever framework shortcut.

**Proportionality.** A small diff should ordinarily produce a small cleanup. Keep modifications attributable to the scoped changes; reading surrounding code is encouraged, rewriting unrelated code is not.

**Apply a fix only when all are true:**
1. It preserves intended behavior, public contracts, error handling, side effects, ordering, and relevant performance characteristics.
2. It removes meaningful complexity, duplication, or wasted work, rather than merely relocating it.
3. It follows established repository patterns and applicable instructions.
4. Its implementation and verification costs are proportionate to the benefit; marginally shorter or merely stylistically different code should be left unchanged.
5. It adds no new dependency or abstraction without a clear net benefit.

Before reviewing, inspect relevant nearby implementations and the repository's applicable instructions (`AGENTS.md`, `CLAUDE.md`, and directory-specific instruction files where present). Follow their precedence and scope; do not impose generic architectural preferences over explicit project conventions. Detect the framework and use its idioms only when they make the code clearer, not simply shorter.

## Phase 1 — Independent reviews

For substantial or complex diffs, launch **four independent subagents concurrently** if the environment supports them. For small, straightforward diffs, review all four angles inline to avoid unnecessary overhead. Send each the same scoped diff (or a reference to it), relevant repository context, and exactly one angle. Ask them to inspect existing code rather than relying on the diff alone. Do not let review agents modify files.

If subagents are unavailable or disproportionate to the scope, perform the four reviews yourself. Do not invent agent results. Do not invent agent results.

Each finding must contain:
- `file` and relevant `line` or range
- concise `summary`
- concrete `cost` (duplication, maintenance burden, wasted work, etc.)
- actionable `suggested_fix`

Only report findings introduced or materially worsened by the scoped changes. Prefer meaningful improvements over stylistic nitpicks.

### Agent 1 — Reuse

Look for new implementations of behavior already provided by the codebase. Search adjacent files, shared utilities, services, and framework conventions. Identify the **specific existing abstraction** to reuse. Do not introduce a shared helper merely to deduplicate two unrelated or trivial expressions. Prefer an existing helper over creating a new one.

### Agent 2 — Simplification

Look for redundant or derivable state, unnecessary indirection, excessive branching or nesting, near-identical code, avoidable configuration, and dead code made obsolete by the change. Propose the smallest clearer equivalent, prioritizing deletion and consolidation before extraction. Evaluate new abstractions by responsibility, likely reuse, clarity at call sites, and total conceptual cost. Avoid clever one-liners and abstraction for its own sake.

### Agent 3 — Efficiency

Look for repeated I/O or queries, redundant computation, unnecessary allocations, avoidable sequential work where operations are truly independent, and added blocking work on hot paths or startup. Check whether long-lived callbacks or objects unnecessarily retain large captured environments; where applicable, retain only required data. Recommend changes only where the cost is plausible and the alternative preserves ordering, concurrency limits, and behavior. Do not optimize speculatively. Preserve observable ordering, transaction boundaries, and resource limits.

### Agent 4 — Altitude (root-cause level)

Check whether the change addresses the underlying mechanism at the appropriate layer rather than adding fragile exceptions or symptom patches. Prefer a small general fix at the owning abstraction when it reduces special cases without expanding scope or changing behavior. Avoid broad architectural rewrites. Respect the project's existing organization (for example, Laravel Actions/Queries or Vue composables where already used); do not invent a new architectural layer.

## Phase 2 — Triage and apply

1. Wait for all reviews to finish.
2. Merge overlapping findings by underlying cause, not just by line number.
3. Validate each finding against the current code; subagent findings are suggestions, not facts. Apply the core decision rules above, and favor the smallest net reduction in complexity.
4. Apply clear, low-risk, behavior-preserving fixes directly, keeping changes narrowly scoped.
5. Skip findings that are false positives, speculative, disproportionately invasive, require substantial out-of-scope changes, or might alter intended behavior. Briefly record why.
6. Preserve existing public interfaces, database query semantics, exception types and error responses, external API contracts, transaction and locking behavior, side effects, observable timing and ordering, and project conventions unless the requested scope explicitly permits otherwise.
7. Do not revert user edits or introduce unrelated formatting churn. Do not commit unless explicitly requested.

## Phase 3 — Verify

Run the most relevant available formatting, static checks, and targeted tests for the edited code. Prefer targeted checks; broaden coverage only when shared infrastructure or broad behavior is affected. Do not add tests merely to justify a trivial refactor, but preserve and update existing tests when needed. If checks are unavailable, too costly, or fail for pre-existing reasons, say so; never claim unrun checks passed. Inspect the final diff for unintended changes, formatting churn, new abstractions, and changes to public contracts. If verification reveals a regression, correct or revert the cleanup rather than leaving the code in a worse state.

## Final response

Keep it brief:
- **Improved:** the concrete changes made, grouped where useful.
- **Skipped:** meaningful findings not applied and why (omit if none).
- **Verification:** checks run and results, or what could not be run.

If no worthwhile improvements are found, say the scoped code is already clean and mention the verification performed.
