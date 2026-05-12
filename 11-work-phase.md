# 11 — Work phase (Phase 2 of each iteration)

## Step 0: Feasibility check FIRST

See `06-feasibility-check.md`. If it returns "block" — do not proceed to
TDD. Comment, label, unassign, move on.

## Step 1: How to actually run tests on this autobox

The autobox does NOT have docker. The pytest binary lives at
`/home/bot/work/meshcore-planner/.venv/bin/pytest`. Tests against the
backend run from `/home/bot/work/meshcore-planner/backend/` with
`PYTHONPATH=.` set. Concretely:

```bash
cd /home/bot/work/meshcore-planner/backend
PYTHONPATH=. /home/bot/work/meshcore-planner/.venv/bin/pytest tests/test_<module>.py -v
```

The venv has ONLY pytest, not the full backend deps. That means:

- **Tests with no third-party imports beyond pytest**: run fine on the
  autobox. Many script-style tests (e.g. `test_sanitize_db_for_bot.py`)
  fall in this bucket. Run them; observe real pass/fail.
- **Tests that import fastapi / sqlalchemy / pydantic / etc.**: will
  fail with ImportError on the autobox. **DO NOT claim such a test
  passes — you literally cannot run it.** Either:
  1. Skip the test in the bot's verification step and document the gap
     in the PR body's Q1-Q5 audit (Q1: "tests requiring fastapi were
     not runnable on the autobox; verified manually that signature is
     unchanged"), OR
  2. Mark the issue `bot-blocked-need-data` (the missing data being
     "venv with full backend deps") and move on.
- **NEVER write `All N tests pass` unless you have CAPTURED the actual
  pytest output in your PR body's audit block.** Hallucinated test
  results are a CRITICAL violation of `02-meshcore-invariants.md`
  (these rules win over your judgment).

## Step 1.5: Ground your branch on current upstream/main

Before creating your work branch, refresh upstream and branch directly
from the tip — do NOT branch from your local `main` (it may be stale by
hours or days, and other contributors merge into upstream constantly).

The cost of skipping this step is the silent-mass-deletion failure
pattern caught on PRs #30, #31, #197, #202 (2026-05-12): the bot's
branch base is N days old, the bot's diff against its own base is
small and looks clean, but `git diff upstream/main..HEAD` reveals the
merge would also delete every file that landed since the branch base.
The Q1–Q5 audit doesn't catch this because the subagent reviews the
bot's intended diff, not the merge result.

```bash
cd /home/bot/work/meshcore-planner

# Verify the workspace is clean. Uncommitted files from a previous
# iteration MUST NOT leak into your new branch — that's how unrelated
# changes ("Discord bot edits in a test(bg_utils) PR") happen.
if [ -n "$(git status --porcelain)" ]; then
    echo "ERROR: working tree dirty before Step 1.5 — workspace contamination."
    git status --short
    # Don't try to clean up automatically — log the dirty state, label
    # the issue `bot-blocked-need-decision`, and exit. AkkerKid investigates.
    exit 1
fi

git fetch upstream main
git checkout -B bot/<issue_number>-<kebab-slug> upstream/main
```

If `git status --porcelain` is non-empty, that's a contamination
incident from a previous iteration. The right move is to log it, block
the issue, and exit — never `git checkout --` your way through it,
because you might be discarding a sibling agent's in-flight work.

## Step 2: Standard TDD

Invoke `superpowers:test-driven-development`. Standard cycle.

Branch naming: `bot/<issue_number>-<kebab-slug>` (already set in Step 1.5).
Commit messages: conventional-commits style, footer `refs #N`.

## Three special rules for this codebase

- Before writing new code, GREP the codebase for existing helpers/
  utilities that solve a similar problem. Extending > duplicating is the
  project preference (see `01-codebase-respect.md`).
- True coordinates (`true_lat`/`true_lon`) NEVER appear in any user-facing
  response. If your work touches user-facing payloads, run the privacy
  test suite (must be runnable on autobox or via a feasibility-block).
- Provenance stamping: every inbound observation needs a Provenance record.
  If you're adding an ingest path, see `backend/app/provenance.py` and
  don't merge without it.

## State file updates — REQUIRED, not optional

You MUST write `/home/bot/.bot-state.md` BEFORE the iteration ends.
The wrapper restarts you in 60 min – 4 h. Without state, the next
iteration has no idea what was in flight.

Use this exact format (overwrite, don't append — keep it short):

```yaml
last_iter_end: 2026-04-28T12:34:56Z
last_status: <claimed|in-flight|blocked|exited-empty>
active_issue: 142            # null if no issue
active_branch: bot/142-foo   # null if no branch
active_step: <free text>     # what step you're on; null if no work
notes:
  - one-line memo per significant action this iter
```

If you exit without writing this file, you've violated the bootstrap
protocol. The reviewer-agent (Plan 4) will treat any PR opened without
a corresponding state-file update as suspicious.

## Commit + push + PR — REQUIRED at end of work phase

Before ending the iteration, you MUST either:
1. **Commit + push the work to the bot's fork**, then call
   `gh pr create` to open the PR (followed by Phase 3 pre-PR review),
   OR
2. **Mark the issue blocked** (`bot-blocked-*` label + comment +
   unassign), and update `.bot-state.md` to reflect the block.

Mid-flight work that hasn't reached either gate should also be
committed (uncommitted modifications are NOT allowed at iteration end —
they pollute the next iter's working tree). Use a WIP commit:

```bash
git commit -am "WIP: refs #N — partial work, will continue next iter"
git push -u origin bot/N-foo
```

The next iteration sees the WIP commit and continues from there.
