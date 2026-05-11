---
allowed-tools: Bash, Read
description: Stage changes, review diff, write a conventional commit message, and commit. Never pushes — user runs gpush manually.
---
# /commit

1. Run `git status` and `git diff --staged` to see what's already staged. Run `git diff` to see unstaged changes.
2. Ask: stage everything, or specific files? Wait for confirmation.
3. Stage as directed.
4. Review the full diff. Check for any .env files, secrets, or credentials accidentally included — stop and flag if found.
5. Propose a conventional commit message (feat/fix/docs/chore/refactor/style) based on what changed. Keep it under 72 chars. Wait for approval or correction.
6. Commit with the approved message. Do NOT push.
7. Remind the user to run `gpush` when ready to push to both GitHub and Forgejo.

Never run git push, gpush, or any remote operation.
