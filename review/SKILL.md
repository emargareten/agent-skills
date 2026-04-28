---
name: review
description: Risk-aware review of a PR, branch, path, or working-tree diff. Find and verify actionable defects; optionally fix with --fix.
argument-hint: "[PR number | branch | path] [--fix]"
---

# Code Review

**Scope → risk map → inspect → verify → report (max 10).** Favor recall when finding candidates and precision in the final report. Do not invent issues. Review only unless `--fix` is explicit.

## 1. Establish scope

- Resolve target: PR against its actual base; branch against merge-base; path restricted to that path. With no target, use PR metadata if available, else default-branch merge-base, upstream tracking branch, then `HEAD~1`. State ambiguous baseline. Never assume upstream is PR base.
- Include unstaged tracked (`git diff HEAD`), staged (`git diff --cached` when not already covered), and relevant untracked files. Avoid double-counting. Inspect changed hunks, enclosing functions, renames, deletions, and affected callers. Report unchanged code only when the change introduces, exposes, or worsens its defect. Report pre-existing defects only when directly relevant to changed behavior.
- Read applicable `AGENTS.md`, `CLAUDE.md`, nested rules, `CONTRIBUTING.md`, and documented standards, respecting scope and precedence.
- Locate a reliable spec: user-provided task, linked issue/PR, matching documented plan, or current-session requirements. Skip spec checks if none; never infer authoritative requirements.
- Risk-map authorization, payments/accounting, persistence/migrations, external effects, jobs, security, concurrency, shared infrastructure, and API contracts. Risk sets depth, not whether files get reviewed.
- For large diffs, batch by coherent area and track coverage. Disclose anything not reviewed.

## 2. Inspect relevant angles

Consider each angle where applicable; spend most effort on correctness, security, regressions, and spec compliance. No need to produce findings for every angle.

1. **Correctness:** Read each changed hunk and surrounding logic. Check conditions, nulls, zero/falsy values, boundaries, validation, authorization, async behavior, encoding, errors, and failure paths. Trace high-risk flows through persistence and external effects.
2. **Removed behavior:** For deleted/replaced guards, branches, validation, tests, or side effects, identify the old invariant and whether replacement preserves it. Check deployment and backward compatibility.
3. **Cross-file/state:** Trace callers, callees, contracts, exceptions, and ordering. Check repeated/concurrent execution, stale state, partial failure, transactions, locks, idempotency, retries, queues, uniqueness, and migrations. Name a concrete failing state or interleaving.
4. **Reuse:** Identify meaningful duplicated logic already handled by a named helper. Avoid abstractions for trivial repetition.
5. **Simplification:** Flag needless state, branches, nesting, wrappers, dead code, unclear names, or speculative generality only with a concrete benefit and safe simpler alternative.
6. **Efficiency:** Check N+1 queries, redundant I/O/computation, avoidable serial work, blocking hot paths, and excessive allocations. Explain when the cost matters; skip micro-optimizations.
7. **Architecture:** Prefer root-cause fixes over brittle special cases. Flag scattered changes or mixed responsibilities only with concrete maintenance impact; avoid broad refactor demands.
8. **Conventions:** Report clear violations of applicable documented rules only. Quote the exact rule and source. Do not impose personal preferences or formatter-only nits.
9. **Spec:** Compare code with reliable requirements and acceptance cases. Identify exact unmet/contradicted requirement, source, affected code, and impact. Material mismatches rank with correctness bugs. Skip when no reliable spec exists.

**Laravel, when applicable:** Check policies, form validation, nullable relations/casts, N+1, `DB::transaction()`, `lockForUpdate()`, jobs before commit, retries/idempotency, migrations on existing data, and money/payment invariants. Do not assume Laravel in other repos.

Review relevant tests. Flag missing coverage only for a specific consequential unprotected behavior. Run targeted existing tests or safe read-only checks when useful; do not modify code during review.

## 3. Verify and prioritize

**Finding quality:** Give an exact file and line/range, preferably on changed code. State a concrete triggering input, state, or sequence and its consequence. Cite supporting code, tests, or relevant requirements; for conventions quote the exact rule and source, and for spec gaps cite the requirement. Suggest the smallest practical fix. Keep severity (impact) separate from confidence (evidence).

- Record candidate **file/line, issue, trigger, consequence, fix, category, severity, confidence**. Retain plausible concrete scenarios for checking, but distinguish confirmed, likely, and needs-verification.
- Deduplicate only the same underlying defect; keep distinct failure modes. Recheck candidates against surrounding code, existing guards, callers, and tests. Drop disproven issues. Label unresolved high-impact concerns as needing verification, not confirmed bugs.
- Rank by severity, then confidence and user impact. Correctness, security, and material spec mismatches outrank cleanup. Report **at most 10**, with no minimum; never manufacture findings.

## 4. Report

- Prefer `ReportFindings` when available. Inspect its actual schema and call **once per review** (a post-`--fix` re-report is the only exception). Set `summary` to the full finding statement and `short_summary` to a title of **≤60 characters**. Supply location, concrete failure scenario, and category where supported. Include suggested fix and confidence **only if the schema supports them**; otherwise include the fix in `summary`, keeping `failure_scenario` focused on the trigger and consequence.
- Map severity by origin and impact: a bug the change introduces or worsens → `important`; a bug already present before the change → `pre_existing`; a minor issue → `nit`. Use the schema's actual enum names; never send an unsupported value. Set top-level `level` to the effort the review ran at — it is not a severity.
- If the schema has `verdict`, set `CONFIRMED` for verified findings and `PLAUSIBLE` for likely or needs-verification ones, stating the remaining doubt in `failure_scenario`.
- Do not duplicate tool-reported findings in prose.
- Otherwise output a concise severity-ordered list: **`file:Lline — severity: problem. Trigger/consequence. Fix.`** Use exact symbols and locations. Include reasoning where needed for security, concurrency, data integrity, or architecture.
- No praise, filler, vague hedging, or process recap. State uncertainty precisely. If clean, say no actionable findings found. Disclose meaningful coverage limits and tests not run; never claim checks ran when they did not.

## 5. Optional `--fix`

Only with explicit `--fix`: finish review first, then apply targeted safe fixes. Preserve intended behavior and unrelated user changes. Skip uncertain, out-of-scope, or behavior-changing fixes and explain why. Run relevant tests/checks and report actual results. If the `ReportFindings` schema has an `outcome` field, call it again with the same findings and set `outcome` to `fixed`, `no_change_needed`, or `skipped` as appropriate before the final summary; otherwise give a short fix-status list. Never silently fix without permission.
