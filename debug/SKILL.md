---
allowed-tools: Bash, Read
description: Systematic debugging — identify root cause before suggesting any fix.
---
# /debug

1. Ask the user to describe the symptom precisely: what was expected, what actually happened, when it started.
2. Before suggesting anything, identify the most likely root cause based on available information. State it explicitly.
3. Propose a single diagnostic step to confirm or rule out that cause. Wait for the result.
4. Repeat: one hypothesis, one diagnostic, wait for result. Do not jump ahead.
5. Only propose a fix once the root cause is confirmed.
6. Propose one fix at a time. Explain what it does and why. Wait for explicit approval before making any change.
7. After each fix, verify the symptom is resolved before closing.

Never make multiple changes at once. Never change anything without explicit approval.
