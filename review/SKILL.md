---
name: review
description: Risk-aware review of a PR, branch, path, or working-tree diff. Find and verify actionable defects; use --deep for parallel review or --fix to apply fixes.
argument-hint: "[PR number | branch | path] [--deep] [--fix]"
---

# Code Review

**Scope → risk map → inspect → verify → report.** Favor recall when finding candidates and precision when reporting. Never invent issues. Review only unless `--fix` is explicit.

## 1. Establish scope

- PR: compare against actual base. Branch: compare against merge-base with intended base. Path: restrict working-tree or targeted diff to that path. No target: uncommitted changes only, on current branch.
- Never assume upstream is PR base. If branch base is unclear, ask when interactive; otherwise use merge-base with default branch and state assumption. If PR base or valid comparison cannot be resolved, disclose blocker and stop.
- Working-tree reviews include staged/unstaged tracked changes (`git diff HEAD`) and relevant untracked files, without double-counting. Explicit PR/branch reviews exclude local changes unless requested. Read supporting code from the correct revision.
- Do not search unrelated branches, worktrees, or commits for additional changes. Empty scope: say so and stop. Scope limits reviewed changes, not supporting code. Report unchanged code only when the change introduces, exposes, or worsens its defect, or the defect is directly relevant to changed behavior.
- Read applicable `AGENTS.md`, `CLAUDE.md`, nested rules, `CONTRIBUTING.md`, and documented standards, respecting scope and precedence.
- Locate reliable requirements: user task, linked issue/PR, matching plan, or current-session requirements. Skip spec checks if none; never invent requirements.
- Risk-map authorization, payments/accounting, persistence, migrations, external effects, jobs, security, concurrency, shared infrastructure, and API contracts. Risk sets depth, not coverage. Judge scale against realistic workload and deployment conditions; disclose material assumptions.
- Batch large diffs by coherent area, track coverage, and disclose omissions.
- Parallel agents only with `--deep` or explicit request, when permitted and available. Assign independent, complementary angles/areas with scope, rules, and spec. Agents review only; primary reviewer verifies candidates, consolidates coverage, and resolves disagreements. Otherwise review directly using the same criteria.

## 2. Inspect relevant angles

Prioritize correctness, security, regressions, and spec compliance. Consider every applicable angle; no finding quota per angle.

1. **Correctness:** Read changed hunks, enclosing logic, renames, and deletions. Check conditions, nulls, falsy values, boundaries, rounding, time zones, validation, authorization, injection, secrets, async behavior, encoding, errors, and failure paths. Trace high-risk flows from input through persistence and external effects to output.
2. **Removed behavior:** Identify invariants behind deleted/replaced guards, branches, validation, tests, and side effects. Verify replacements preserve intended behavior, deployment compatibility, and existing data.
3. **Cross-file/state:** Trace callers, callees, contracts, exceptions, and ordering. Locate all affected callers when signatures or behavior change. Check concurrency, repeated execution, stale state, partial failure, transactions, locks, idempotency, retries, queues, uniqueness, and migrations. Identify concrete failing states or interleavings.
4. **Reuse:** Identify meaningful duplication; cite existing helpers. Prefer existing code, native language, standard library, and framework capabilities over custom implementations or dependencies. Avoid abstractions for trivial repetition.
5. **Simplification:** Flag needless state, branches, wrappers, nesting, dead code, unclear names, unused options, and speculative generality only with concrete benefit and safe alternatives. Prefer deletion. Merge logic that must change together. Split by responsibility, not length; avoid cosmetic extraction. Fewer lines alone are not an improvement.
6. **Efficiency:** Check N+1, redundant I/O/computation, avoidable serial work, blocking hot paths, excessive allocations, unbounded growth, and repeated work across processes. Explain impact at realistic load; skip micro-optimizations and speculative scaling.
7. **Architecture:** Prefer root-cause fixes. Flag scattered logic or mixed responsibilities only with concrete impact. Avoid unnecessary layers, broad refactors, and speculative abstractions. Respect documented shortcuts unless assumptions fail or material risks remain.
8. **Conventions:** Check applicable documented rules only. No personal preferences or formatter-only nits.
9. **Spec:** Compare against reliable requirements and acceptance cases. Material mismatches rank with correctness bugs.

**Laravel, when applicable:** Check policies, form validation, nullable relations/casts, N+1, `DB::transaction()`, `lockForUpdate()`, jobs before commit, retries/idempotency, migrations on existing data, and money/payment invariants.

Review relevant tests, including unchanged coverage. Flag missing tests only for consequential unprotected behavior. Prefer meaningful regression tests over coverage targets. Run targeted existing tests or safe read-only checks when useful and permitted. Review without editing.

## 3. Verify and prioritize

**Finding quality:** Exact file/line, preferably changed code; concrete trigger, consequence or maintenance cost; supporting evidence; smallest practical fix. For conventions, quote exact rule and source; for spec gaps, cite exact unmet requirement and source.

- Keep severity (impact), confidence (evidence), and origin (introduced, exposed, worsened, pre-existing) separate.
- Recheck surrounding code, guards, callers, tests, and actual framework behavior. Confirm failing states are reachable, unused code is unused, and fixes preserve intended behavior without introducing new risks.
- Deduplicate underlying defects, not distinct failure modes. Drop disproven, speculative, and non-actionable issues. Report unresolved concerns only when concrete and consequential; state what needs verification.
- Rank by severity, then confidence and user impact. Correctness, security, data integrity, and material spec gaps outrank cleanup. Report at most 10, no minimum; exceed only for distinct, verified critical defects.

## 4. Report

- Prefer `ReportFindings` when available. Inspect schema; call once (post-fix excepted). Full finding in `summary`, title ≤60 characters in `short_summary`, trigger/consequence in `failure_scenario`. Include location, category, fix, and confidence where supported; otherwise include fix in `summary`.
- If the schema uses `important`, `pre_existing`, and `nit`, map introduced/exposed/worsened material defects to `important`, unaffected relevant pre-existing defects to `pre_existing`, and minor issues to `nit`. Preserve internal impact ranking. `level` means review effort.
- If supported, use `CONFIRMED` for verified findings, `PLAUSIBLE` for unresolved findings with explicit uncertainty. Use supported fields/enums only. Do not duplicate tool findings in prose.
- Otherwise use a severity-ordered numbered list: `file:Lline — severity: problem. Trigger/consequence. Fix.` Use exact symbols and locations. Label pre-existing defects and uncertainty. Keep findings terse; expand when complex reasoning requires it.
- No praise, filler, vague hedging, or process recap. If clean, say no actionable findings found. Disclose meaningful coverage limits, workload assumptions, and tests not run. Never claim checks ran when they did not.

## 5. Optional `--fix`

Only with explicit `--fix`, finish review before applying targeted safe fixes.

- Preserve intended behavior, comments, and unrelated changes. Prefer minimal diffs and deletion.
- Verify target; never switch branches, modify another worktree, or edit a different target without permission.
- Skip uncertain, out-of-scope, or product-decision fixes; explain why.
- Add/update regression tests where appropriate; never weaken assertions to pass. Run relevant checks, recheck diff, and report actual results.
- If supported, re-report the same findings with `outcome`: `fixed`, `no_change_needed`, or `skipped`; otherwise summarize fix status.
