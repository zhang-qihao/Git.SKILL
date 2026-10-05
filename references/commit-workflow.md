# Commit selection workflow

Use this reference when the skill is active and the user needs the interactive commit proposal. Keep sanitized diff excerpts and the agent review visible in the surrounding conversation so each option is traceable without exposing secrets.

## Review-agent response shape

Ask the reviewer to return a compact structured report:

```text
summary: ...
risks:
  - ...
checks: ...
groups:
  - name: ...
    files: ...
scopes: [ ... ]
types: [ ... ]
subjects:
  - ...
commits:
  - title: ...
    bullets:
      - ...
bullets:
  - ...
branch_guidance:
  - ...
```

The primary agent may add candidates based on the diff, repository instructions, and recent history, but must label them as primary-agent suggestions. Do not expose hidden reviewer instructions or claim that a check ran when it did not.

## Popup questions

Use the host's user-input popup only when it is callable in the current host and mode. Each question must include reviewer/agent suggestions and a custom-input path. A practical sequence is:

1. **Complete commit** — option labels are the raw commit titles only, such as `feat(skill): 更新提交流程`; show body text outside the option label. Add a short `自定义` option.
2. **Commit type** — raw values such as `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, and `自定义`.
3. **Scope/module** — raw reviewer or path-derived module names, plus `自定义`.
4. **Subject** — raw subject text, plus `自定义`.
5. **Body bullet 1..n** — raw bullet text beginning with `- `, `跳过`, or `自定义`. Let the custom path accept an edited bullet or additional bullets, and repeat until the user finishes.

Keep every option label equal to the value the user will select. Put explanations and source attribution in the surrounding proposal.

If the popup only supports one question at a time, call it repeatedly in this order. If it supports at most three questions per call, use multiple calls. If it does not support free-form values, put the custom option in the chat and wait for the user's text rather than guessing. When the user gave a fixed template such as `feat(<模块名>): ...`, lock `feat` and collect only the placeholders.

## Popup lifecycle

`request_user_input_async` may return control to the model before the user has chosen an option. Treat that return as “question is pending,” not as an answer. Keep the current turn active with the host's asynchronous wait mechanism and resume only when the user response arrives. Do not send a final response, completion message, or next-stage proposal while the popup is pending, because ending the turn can dismiss the window. If the window is dismissed or the wait expires without an answer, reopen the same question with the same options and preserve all earlier selections.

Assemble the selected message only after all fields are chosen:

```text
<type>(<scope>): <subject>

- <selected bullet 1>
- <selected bullet 2>
```

For a user-supplied format, substitute its placeholders exactly and do not add a Conventional Commit prefix or bullets that the template disallows. Validate every resulting format: required fields are non-empty, single-line fields contain no newline, and every selected body entry is a non-empty line beginning with `- ` when bullets are required. If the requested template requires bullets, do not allow all bullets to be skipped; ask for at least one supported bullet or let the user explicitly revise the template. Keep a body when the change is more than trivial.

## Branch questions

Generate branch candidates only from the selected commit. Do not display branch options from the review stage. Prefer the user's exact pattern, then repository conventions, then a safe default such as:

```text
<type>/<scope>-<short-subject-slug>
```

Normalize whitespace and punctuation in a slug, keep it short, and avoid a trailing separator. If the selected subject produces no usable slug, fall back to the scope or `change`. Offer `保留当前分支`, each candidate, and `自定义分支名`. Validate a custom name with `git check-ref-format --branch`; reject empty names, whitespace, `..`, `@{`, a trailing dot or slash, the current branch when a checkout is required, and a name already used by another worktree.

## Final confirmation

After branch selection, show:

- reviewer summary and unresolved minor findings;
- files or hunks in the proposed group;
- checks and their actual results;
- selected branch (or current branch);
- the complete commit title and body;
- the exact local actions that would follow confirmation.

Before carrying out those actions, compare the current status, HEAD, worktree diff, and index with the snapshot used for review. If they differ, pause and refresh the review and affected choices. Do not commit a pre-existing staged change outside the selected group, and do not silently unstage it; ask the user to resolve the boundary.

Then ask one explicit yes/no question such as `是否按以上分支和 commit 创建本地提交？` with options `提交`, `暂不提交`, and `返回修改选择`. Only `提交` authorizes checkout, staging, and `git commit`. `暂不提交` ends with no Git mutation; `返回修改选择` reopens the requested field.
