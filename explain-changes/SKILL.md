---
name: explain-changes
description: Briefly explain the purpose of code changes and why each changed file needed modification, grouping files that share the same reason. Use when asked to summarize a diff, explain changes, or explain why files changed.
argument-hint: "[PR number | branch | commit | file path]"
---

# Explain Changes

Explain **what changed and why**, in plain, concise language. This is a summary, **not a code review**: do not hunt for bugs, suggest refactors, modify files, or run tests unless explicitly requested.

## 1. Identify the scope

- If an argument specifies a PR, branch, commit, or file path, inspect that target using the available Git/hosting tools.
- Otherwise, determine the actual PR target branch when available and compare from its merge base. If unavailable, use the repository default branch, then a suitable upstream tracking branch, then `HEAD~1` as a last resort. Do not assume upstream is the PR base.
- Include uncommitted tracked changes (`git diff HEAD`) when present, and staged changes as part of that diff. Avoid double-counting overlapping changes.
- Identify untracked files with `git status --short`; inspect relevant new files if they are part of the requested work.
- Use `git diff --stat` and `git status --short` to establish the full file list. For renamed files, examine the rename and content changes separately.
- If the requested diff is empty, say so rather than inventing changes.

## 2. Understand the intent

- Read the diff and enough surrounding code to understand each meaningful change.
- Determine the overall goal and the specific reason each file needed modification.
- Distinguish **observed changes** from **inferred intent**. Do not claim to know the author's motivation if the diff does not establish it.
- Lead with user-visible or functional behavior and purpose, not mechanical details. Mention implementation details only when they explain why the change was necessary.
- Account for every changed file, prioritizing functional impact and grouping files that serve the same purpose (e.g., migration + model + tests). Avoid repeating the same explanation.
- Call out unrelated changes as a separate group instead of forcing them into one narrative.

## 3. Output format

Use this structure, keeping it short:

### Summary
One to three sentences describing what the changes accomplish and, when evident, the problem they address.

### Files changed
- **`path/to/file.php`** — Why this file needed to change and its role in the solution.
- **`path/to/other.php`, `path/to/test.php`** — Shared reason for these changes.

Group by purpose when helpful, using a short bold group label. Keep each item to one sentence whenever possible. For large diffs, prefer grouped explanations over long per-file prose, but ensure all changed files are accounted for.

## Rules

- Be concise, direct, and easy to understand; avoid generic phrases like “updated logic” or “improved code.”
- Explain **why**, not merely **what**. Example: “Added a unique constraint to prevent duplicate refunds,” not “Changed the migration.”
- Do not speculate about benefits or requirements not supported by the diff.
- Do not report bugs, give approval, or propose improvements unless asked.
- Do not paste the diff or include a line-by-line walkthrough.
- If the changes have multiple unrelated goals, summarize each separately.
- If a file's purpose cannot be determined confidently, state that briefly.
- Use the user's language unless instructed otherwise.
