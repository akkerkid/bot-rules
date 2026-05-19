# 09 -- Commit hooks + lint CI (meshcore-planner)

A `git commit` in meshcore-planner runs two independent checks:

1. **code-check** (Claude PreToolUse hook, `scripts/lint/code-check-precommit.sh`)
   -- magic numbers, sub-5-char identifiers, banned imports, banned regex
   patterns, non-ASCII. Fix-or-explain: fix each finding or justify it.

2. **diff-aware lint** (`.githooks/pre-commit` -> `scripts/lint/run.sh`)
   -- ruff, darker, eslint, and the async-blocking check, on changed lines
   only. Hard-fail: the commit is blocked until the findings are resolved.

The same diff-aware harness runs in CI (`.github/workflows/lint.yml`) on every
PR -- so even if a local hook is not installed, your PR gets the lint check.

If either reports findings, fix each one or justify it in your PR body as an
intentional exception. Do not `--no-verify` past them without explicit
authorization.
