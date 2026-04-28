# 11 — Work phase (Phase 2 of each iteration)

## Step 0: Run feasibility check first

See `06-feasibility-check.md`. If it returns "block" — do not proceed to
TDD. Comment, label, unassign, move on.

## Step 1: Standard TDD

Invoke `superpowers:test-driven-development`. Standard cycle.

Branch naming: `bot/<issue_number>-<kebab-slug>`
Commit messages: conventional-commits style, footer `refs #N`.

## Three special rules for this codebase

- Before writing new code, GREP the codebase for existing helpers/
  utilities that solve a similar problem. Extending > duplicating is the
  project preference (see `01-codebase-respect.md`).
- True coordinates (`true_lat`/`true_lon`) NEVER appear in any user-facing
  response. If your work touches user-facing payloads, run the privacy
  test suite: `pytest backend/tests/test_privacy.py`.
- Provenance stamping: every inbound observation needs a Provenance record.
  If you're adding an ingest path, see `backend/app/provenance.py` and
  don't merge without it.

## State file updates

Update `/home/bot/.bot-state.md` whenever you complete a meaningful step:

```yaml
active_issue: 142
active_branch: bot/142-foo
active_step: tests-written
last_update: 2026-04-27T15:23:00Z
```

Hit the iteration timeout? That's fine. State file picks up next iter.
