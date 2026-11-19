---
name: simplify
description: Simplify changed code through reuse, reduced complexity, efficiency, and root-cause design. Apply safe, behavior-preserving improvements and verify them; not a bug review. Use when asked to simplify or clean up recent changes; not for requested redesigns or large refactors.
argument-hint: "[PR number | branch | commit/range | file/directory]"
---

# Simplify Changed Code

**Scope → inspect four angles → validate → improve → verify.** Improve maintainability, not correctness. Do not hunt for bugs or perform a security review. Apply worthwhile low-risk cleanups by default; do not commit unless requested.

## 1. Establish scope

- Accept a PR, branch, commit/range, or file/directory. For PRs compare against the actual base; for branches use the merge-base; for a commit compare against its parent (root commit: against the empty tree), and for a range compare its endpoints. A file/directory restricts scope to that path (alone: uncommitted changes within that path; with a PR/branch/commit/range target: that diff within that path). With no target, use uncommitted changes only, on any branch. Do not search unrelated branches, other worktrees, or past commits for changes unless explicitly targeted. Do not assume upstream is the PR base.
- Inspect `git status --short`. Include relevant staged, unstaged (`git diff HEAD`), and untracked changes without double-counting. If the entire resolved scope (any explicitly targeted diff plus relevant uncommitted changes) is empty, say so and stop. Inspect changed hunks and surrounding code, callers, tests, and existing helpers.
- Apply edits only to the current working tree. For historical commit/range targets, skip candidates whose code has since changed. Limit edits to changed code and directly related code. Read wider context as needed; do not refactor unrelated legacy code or overwrite user changes.
- Read applicable repository instructions (`AGENTS.md`, `CLAUDE.md`, nested/path-scoped rules, and documented standards). Respect scope and precedence. Follow existing framework conventions rather than imposing new architecture.

## 2. Decide whether to change

**Deletion first; abstractions last.** Prefer removing unnecessary code or state, then reusing existing code, before adding helpers, layers, or dependencies. Optimize for fewer concepts and easier maintenance, not shortest code.

Apply a change only when it:

- Preserves intended behavior, public contracts, errors, side effects, observable ordering/timing, transaction and lock semantics, and does not make performance meaningfully worse.
- Removes meaningful duplication, complexity, or wasted work instead of moving it elsewhere.
- Fits repository conventions, with benefit proportionate to implementation and verification cost.
- Introduces no new abstraction or dependency without clear net benefit.

**Doing nothing is valid.** If behavior preservation cannot be established with reasonable confidence, skip the change. Leave trivial duplication, speculative optimizations, subjective style changes, and uncertain or risky refactors alone. Small diffs ordinarily warrant small cleanups.

## 3. Inspect four angles

Review all four angles inline, allocating effort according to the diff. Validate candidates against surrounding code before acting.

1. **Reuse:** Find new code duplicating a specific existing helper, utility, service, or framework capability. Name what to reuse. Avoid extracting trivial or unrelated repetition.
2. **Simplification:** Find derivable state, redundant branches, needless nesting or wrappers, obsolete code, avoidable configuration, and speculative generality. Prefer deletion and consolidation over extraction; preserve readability.
3. **Efficiency:** Find repeated I/O/queries, redundant computation, avoidable allocations, unnecessary serial work, and blocking hot paths. Optimize only with a plausible cost and a safe alternative; preserve ordering, concurrency limits, and resource behavior.
4. **Root-cause design:** Identify symptom patches or scattered special cases that belong in an existing owning abstraction. Prefer the smallest root-level improvement without broad redesign or new architectural layers.

For each candidate, identify its **file/location, concrete cost, and smallest safe improvement**. Consider only issues introduced or materially worsened by the scoped changes.

## 4. Validate and apply

- Deduplicate candidates by underlying cause. Recheck each against existing code and tests; reject false positives, speculative benefits, and disproportionately invasive fixes.
- Apply clearly beneficial, low-risk, behavior-preserving changes directly. Skip uncertain changes and briefly explain meaningful omissions.
- Preserve unrelated edits and comments. Avoid unrelated formatting churn. Never silently change public interfaces, database semantics, external contracts, exception behavior, or side effects.

## 5. Verify

- Run relevant targeted tests, formatting, and static checks; broaden only when shared behavior is affected. Follow repository-required checks. Update tests only to reflect internal implementation changes, never to change expectations about observable behavior; if a cleanup fails such an assertion, revert the cleanup instead. Do not add tests solely for trivial cleanup.
- Inspect the final diff for unrelated edits, new complexity, and unintended behavior changes.
- If a cleanup causes a regression or check failure, fix or revert that cleanup. Distinguish pre-existing failures from new ones. Never claim unrun checks passed.

## 6. Report

Briefly state **improvements made**, **meaningful changes skipped and why** (if any), and **verification results or limitations**. If no worthwhile improvements exist, say so. No praise, filler, or review-process recap.
