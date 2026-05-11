---
allowed-tools: Bash, Read
description: Review staged or recent changes before committing. Checks for correctness, secrets, and style issues.
---
# /review

1. Run `git diff --staged`. If nothing staged, run `git diff HEAD`.
2. Read any files that changed in full if the diff is hard to interpret.
3. Check for:
   - Hardcoded credentials, API keys, tokens, or passwords
   - Broken logic or obvious bugs
   - Inconsistent style with the surrounding code
   - Commented-out code that shouldn't be committed
   - Debug print statements or temporary workarounds
   - Files that shouldn't be tracked (.env, *.key, *.pem)
4. Give a verdict: PASS (safe to commit) or FAIL (issues found).
5. For FAIL: list each issue clearly with the file and line. Do not attempt fixes — let the user decide.
