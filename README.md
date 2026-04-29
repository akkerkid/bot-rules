# bot-rules

Read-only directives for the autonomous MeshOMatic bot. Pulled at the
start of every Ralph loop iteration before any work begins.

| File | Purpose |
|------|---------|
| 00-style.md | Programming style |
| 01-codebase-respect.md | Extend > duplicate; trace 1-2 callers up/down |
| 02-meshcore-invariants.md | CODEX hard invariants (mirror) |
| 03-self-awareness.md | What the bot can and cannot do |
| 04-tools-and-skills.md | Which Claude Code skills + model selection |
| 05-respect-locks.md | Issue-assignee deconfliction |
| 06-feasibility-check.md | Pre-flight: data / hardware / decision |
| 07-no-secrets.md | Never write secret values to any file or message |
| 08-rebase-and-retest.md | When main has moved under your PR (label `bot-rebase` or `@meshomatic please rebase` comment) |
| 10-inbox-phase.md | Phase 1 of each iteration: find work |
| 11-work-phase.md | Phase 2: TDD on the chosen issue |
| 12-pre-pr-review.md | Phase 3: fresh-context subagent audit |

Edits here take effect within one bot iteration cycle (no autobox redeploy).

Spec: https://github.com/akkerkid/meshcore-planner — `docs/superpowers/specs/2026-04-27-autonomous-meshomatic-bot-design.md`
