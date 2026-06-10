---
name: commit
description: Review staged changes and propose a conventional commit message for user approval before committing
disable-model-invocation: true
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git add *) Bash(git commit *) Bash(git log *)
---

## Your task

1. Run `git status --short` and `git diff HEAD` to review what changed.
2. Propose a conventional commit message (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, etc.) that is concise and focused on the change.
3. **Show the proposed message to the user and ask for approval before committing.** Do not commit until the user confirms.
4. Once approved, run:
   - `git add .` to stage all changes
   - `git commit -m "<approved message>"`
   - `git log -1 --oneline` to confirm success

## Rules

- Never commit without explicit user approval.
- Never include "Generated With Claude", "Co-Authored-By: Claude", or any AI attribution in commit messages.
- Keep commits focused — one feature or fix per commit.
- If the diff spans unrelated changes, flag this and suggest splitting into separate commits.
