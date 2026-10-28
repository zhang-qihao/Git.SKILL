---
name: git-smart-commit-zqh
description: Review Git changes, split them into logical Conventional Commits, and use popups to select two message candidates per commit, one branch for the batch, and local commit, commit-and-push, or no commit. Use when preparing or submitting Git changes with user-selected messages and branch.
---

# Git Smart Commit ZQH

Review and prepare the whole batch before changing Git state. All selections must use user-input popups, with free-text editing available. If popups are unavailable, explain the limitation and pause; never substitute chat choices or select automatically. When using an asynchronous popup, keep the turn active and wait for the response; do not end the turn while it is pending. A pending or dismissed popup is not approval. Preserve collected answers and resume the unanswered step when the user responds or asks to continue.

## Workflow

1. **Inspect and group.** Read project instructions, current branch/HEAD, remotes/upstream, staged and unstaged diffs, and untracked files. Stop if there is nothing to commit. Review risks and run applicable repository checks without auto-fixing source files; report failed or unavailable checks and do not mark a proposal ready when required checks fail. Group changes by independent purpose, keeping implementation and its tests together; split files by hunks when needed. Order dependent groups so intermediate commits remain coherent. Keep a single-purpose change as one commit. Show each group's purpose, file/hunk boundary, dependencies, and checks. Ask about ambiguous scope instead of guessing; do not edit code to enable splitting.
2. **Choose each message.** For each group, show exactly two complete, accurate Conventional Commit candidates, A and B, including any body/footer. Both describe the same group. Use a popup with the two titles as options and free-text input for edits. Accept a complete candidate without asking again for individual fields. Validate custom messages too; retain other selections when revising one group.
3. **Choose one branch for the batch.** After all messages are selected, open one popup with exactly two options: `保留原分支：<actual-current-branch>` and `创建新分支：<generated-name>`. Generate the new name from the batch's main purpose using the rules below. Allow a custom new name through free text. Check validity and local/remote name collisions before offering it; check worktree occupancy when relevant. In detached HEAD, explain that keeping a branch is unavailable and resolve the destination through a popup before proceeding.
4. **Choose the operation.** Show the selected branch, ordered complete messages and file/hunk boundaries, checks/risks, and the proposed push remote and target branch. Identify any earlier unpushed commits that a push would also publish. Resolve an ambiguous push destination through a popup before authorizing push; a missing remote must not block local-only commits. Then open a popup with exactly: `提交本地`, `提交本地并 push 远程`, `不提交`. The selection authorizes the displayed batch and operation; do not ask for duplicate confirmation. `不提交` leaves branch, index, worktree, and remote unchanged.
5. **Execute and verify.** Only after selection of a commit operation, recheck HEAD, branch, index, and worktree against the reviewed snapshot. Revisit only affected choices if content changed. Preserve unrelated edits and pre-existing staging; if the index boundary cannot be preserved safely, pause for clarification. Create/switch to the selected new branch without force if needed, and verify its exact name. For each group, stage only its confirmed files/hunks, inspect the full staged diff, run `git diff --cached --check`, and commit with the selected message via a message file or argument-safe invocation. Verify each commit's message and content. If push was selected, recheck the publication scope and obtain popup approval for any change before proceeding, then push only the agreed branch to the agreed remote without force; establish upstream for a new branch. Verify the remote branch tip matches the local tip. Report commit hashes/titles, actual branch, push result if requested, remaining changes, and unresolved checks. On failure, stop and report completed work; do not automatically undo commits or rewrite history.

## Commit rules

Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) strictly, including for user edits and repository templates:

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

- Use `feat` for a new feature and `fix` for a bug fix. Other accurate types include `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `chore`, and `revert`.
- Require a type, colon followed by a space, and a nonempty concise description. Scope, if present, is a noun describing the code section in parentheses. Default to lowercase type/scope and Chinese descriptions; honor another requested language.
- Body and footers are optional, separated by blank lines. Bodies are free-form, not mandatory bullet lists. Footers use valid Conventional Commits token/separator syntax.
- Mark breaking changes with `!` immediately before `:` or an uppercase `BREAKING CHANGE:` footer explaining the break. With `!` alone, the description must explain the break. Accept the standard's synonymous `BREAKING-CHANGE:` footer.
- Do not require scope, impose an arbitrary title-length limit, invent changes, or let a custom template bypass the standard.

## New branch rules

Use exactly `<type>/<short-description>`. This is the skill's convention; [Git](https://git-scm.com/docs/git-check-ref-format) defines name validity, not these semantic categories.

- Type: `feat|fix|refactor|docs|test|perf|build|ci|chore|revert`, chosen for the batch's main purpose; individual commits retain their own types.
- Description: lowercase English words/digits separated by single hyphens, matching `[a-z0-9]+(-[a-z0-9]+)*`. Example: `feat/login-session`.
- Exactly one slash; no `codex`, username, or extra prefix. Pass the confirmed name unchanged to Git and run `git check-ref-format --branch` before creation. Recheck collisions immediately before creating; never silently reuse or rename a colliding branch.
- Existing branch names are exempt when explicitly retained. Do not rename or delete them automatically. If a tool forces a different prefix, use an authorized alternative that preserves the name or stop and explain.

## Boundaries

Never expose secrets in review excerpts or mix unrelated changes, secrets, local environment files, or generated logs into a commit. Never use blanket staging, overwrite user edits, force-switch, reset, force-push, bypass hooks/checks, or rewrite history. This workflow does not authorize code fixes, dependency installation, automatic pull/rebase/merge, tags, or PR creation. If a hook changes the staged content, re-review the affected group before continuing.
