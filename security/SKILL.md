---
allowed-tools: Bash, Read
description: Security check before pushing or deploying. Scans for exposed secrets, insecure patterns, and permission issues.
---
# /security

1. Run `git diff HEAD` and scan all changed files for:
   - API keys and tokens: Anthropic API keys, AWS access keys, GitHub tokens (ghp_…/gho_…), Cloudflare tokens (CF_…), PEM-encoded private keys
   - Hardcoded IPs or credentials that belong in .env
   - World-writable file permissions (check with `ls -la`)
   - Shell scripts without `set -e` or `set -eo pipefail`
   - curl | sh or wget | bash patterns
2. Check .gitignore covers: .env*, *.key, *.pem, secrets/, credentials/, .ssh/
3. Run `git status` and flag any untracked .env or secrets files that haven't been gitignored.
4. Give a verdict: PASS or FAIL with specific issues listed.
5. Do not attempt to fix anything — flag and let the user decide.
