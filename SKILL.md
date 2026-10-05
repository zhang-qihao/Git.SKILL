---
name: git-smart-commit-zqh
description: Review a Git worktree with a dedicated agent, then guide the user through editable commit and branch choices before any local commit or publication. Use when the user asks to commit, submit, push, or prepare Git changes and wants the commit format or branch naming under their control.
---

# Git Smart Commit ZQH

Use this skill as a staged decision workflow for the current Git worktree. The user may provide a commit or branch template in the request; that template has priority over repository conventions and the defaults below. Do not modify the worktree while gathering review results or choices.

## Required sequence

1. **Inspect the worktree.** Read the repository root, current branch, remotes, status, recent commit style, staged and unstaged diffs, and the relevant project instructions. If there is no change to commit, report that and stop. Identify unrelated, secret, generated, or environment files and keep them out of any proposal unless the user explicitly includes them.
2. **Always attempt to start exactly one review agent before proposing a commit.** Give it sanitized status and diff context plus repository instructions (include full content only when it is safe), and tell it not to edit, stage, commit, switch branches, push, or open a PR. Ask it to return: a change summary, risks or missing tests, logical commit groups, affected module/scope candidates, complete commit candidates, commit type and subject candidates, one candidate for each body bullet, and branch-naming guidance for use after the commit is chosen. Treat its output as review input, not as user approval. If delegation is unavailable, state that limitation and perform the same review yourself before presenting choices.
3. **Run relevant read-only checks.** Discover repository-provided test, lint, typecheck, build, or formatting commands that apply to the changed code. Run appropriate checks before the final proposal. Do not install dependencies or repair unrelated failures without the user's authorization. Report failures and do not present a ready-to-commit proposal when a required check is failing.
4. **Choose the commit first.** Present the review findings and first offer 2–3 complete commit candidates from the reviewer and primary agent, with the option label showing only the raw commit title (for example, `feat(skill): 更新提交流程`). Show each body separately in the surrounding proposal. Add a short custom option such as `自定义`. Then let the user keep a candidate or edit it through these fields in order:
   - commit type: reviewer candidates, repository-compatible Conventional Commit types, and a custom option;
   - functional scope/module: reviewer and path-based candidates, plus a custom option;
   - subject: reviewer candidates and a custom option;
   - body bullets: for each proposed bullet, let the user select the reviewer's wording, edit it, add another bullet, or skip it. Keep the final body in the requested format, with one `- ` line per selected bullet.

   If the user supplied a literal type, scope, or template, preserve its fixed parts and ask only for its placeholders unless the user explicitly asks to change them. A user-supplied format takes precedence over the Conventional Commit defaults.

   The user must be able to choose a reviewer suggestion or enter their own value at every field. Use each selected value itself as the option label, and place source or explanatory context in the surrounding proposal. Use `request_user_input_async` only when it is callable in the current host and mode; keep each popup within its supported option limit. After opening an async popup, keep the turn alive and wait for its response; do not send `final`, announce completion, or otherwise end the turn while a selection is pending. If the host closes the popup before a response arrives, reopen the same question and preserve the already collected choices. If the UI tool is unavailable, show the same numbered choices in chat and wait for the user's reply; never silently choose an option. Do not ask for branch names until the commit fields are settled. See [references/commit-workflow.md](references/commit-workflow.md) for the interaction schema and validation rules.
5. **Generate and choose a branch after the commit is settled.** Derive 2–3 concise branch candidates from the selected type, scope, and subject, following any user-provided branch template. If no template was provided, use the repository's established prefix; otherwise use a safe default such as `<type>/<scope>-<slug>` and explain that it is a default. Include an option to keep the current branch and an explicit custom branch option. Check for name collisions and worktrees using the selected branch before any checkout.
6. **Show the final summary and ask for confirmation.** Display the review conclusion, changed-file grouping, checks and their results, selected branch, and the complete commit message. Ask separately whether to create the local commit. A branch choice is not permission to checkout, stage, commit, push, or open a PR. If the user requests edits, return only to the relevant choice step and preserve the other selections.
7. **Only after an explicit commit confirmation, mutate Git.** Re-read status and compare the current HEAD, worktree diff, and index with the review snapshot. If files or hunks changed, return to the affected review or choice stage. Preserve unrelated edits. Check that the full index contains only the confirmed group; if pre-existing staged content falls outside it, pause and ask the user to resolve the staging boundary rather than silently unstage or overwrite anything. If a new branch was selected, verify it with `git check-ref-format --branch`, check collisions and worktrees again, and switch without force or data loss. Stage only the confirmed file/hunk group, review `git diff --cached`, run `git diff --cached --check`, and create the selected commit using a message file or argument-safe invocation rather than shell interpolation. Then report the commit hash and remaining status. Never push or open a PR unless the user separately authorizes the exact remote, branch, and PR target.

For multiple logical commit groups, repeat the commit-choice and branch-choice stages for each group, showing the proposed file/hunk boundary before asking for confirmation. Do not use `git add -A` unless the user explicitly confirms that every worktree change belongs to the current group.

## Default commit format

When the user has not supplied a format and repository instructions do not define one, use Conventional Commits with a required functional scope:

```text
feat(<模块名>): 改动信息

- 改动1
- 改动2
```

Offer accurate types such as `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`, and `revert`; do not force `feat` when the diff indicates another type. Keep the scope concise and lower-case unless the user's template says otherwise. Keep the subject concrete and concise. Use Chinese body bullets by default, preserving the user's language or exact template when they specify one. Do not invent a bullet that the diff does not support.

## Safety boundaries

- Never include secrets, local environment files, generated logs, or unrelated user edits without explicit confirmation.
- Never rewrite existing history, reset files, force-switch branches, or overwrite uncommitted changes.
- Never infer permission to commit from permission to review, or permission to push from permission to commit.
- Keep read-only checks from auto-fixing or writing into the source tree when possible; use temporary output locations and verify the worktree afterward.
- Redact secrets from any diff excerpt shown to the user or reviewer while preserving file and hunk context.
- Report exactly which checks ran, what the reviewer found, which files were staged, and which commit was created.
