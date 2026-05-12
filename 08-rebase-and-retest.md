# 08 — Rebase and re-test (when main moves under your PR)

Other contributors merge into `akkerkid/meshcore-planner` `main` while your PR is awaiting review (or, increasingly, while your iteration is in flight — sibling agents merge dozens of PRs per day). When that happens, your branch may now conflict with main, OR your tests may have been invalidated by changes to code you depend on. There are TWO triggers for this rule:

**Trigger A — proactive (you, before `gh pr create`):**

Before opening the PR, run the two-dot stat check:

```bash
git fetch upstream main
git diff upstream/main..HEAD --stat | tail -1
```

The `..` (two-dot) syntax shows what would actually land if you merged. If the totals are larger than what you intended to ship — extra files touched, large `-` deletion counts — your branch base is stale, and `mergeStateStatus: CLEAN` is lying to you (git auto-resolves add-vs-delete by deleting). DO NOT open the PR. Run the rebase-and-retest procedure below first. Forensic notes from the 2026-05-12 incident (PRs #30, #31, #197, #202) all caught this way: bot opens PR with `+729/-5` claim; actual merge against current main was `+1604/-14806`.

Heuristic: if the two-dot file count > 1.5× the in-scope files you touched, OR the two-dot deletion count > your insertion count, you're stale. Rebase.

**Trigger B — reactive (someone tells you):**

AkkerKid or the reviewer-agent signals this by:

- Applying the **`bot-rebase`** label to your PR, AND/OR
- Posting a comment on the PR containing the literal phrase **`@meshomatic please rebase`** (case-insensitive)

Either signal puts the PR at the top of your inbox queue (see `10-inbox-phase.md` step 1.5). Your job:

## The rebase-and-retest procedure

### Step 1: Read the trigger

```bash
gh pr view <N> --repo akkerkid/meshcore-planner --json labels,comments
```

If labelled `bot-rebase` and there's a comment with specific guidance, read the comment first. The reviewer may have noted what they think will conflict, or what behavior they want preserved.

### Step 2: Check out the PR's branch

```bash
cd /home/bot/work/meshcore-planner
git fetch upstream
git fetch origin
git checkout <branch-name>           # branch from gh pr view
```

### Step 3: Attempt the rebase

```bash
git rebase upstream/main
```

**If the rebase succeeds cleanly** (no conflicts), proceed to Step 4.

**If conflicts**: read each one. Three cases:

- **Trivial whitespace/import-order conflicts**: resolve them in the obvious way, `git add`, `git rebase --continue`. Document the resolutions in the PR comment in Step 6.
- **Semantic conflicts** (your code calls a function that was renamed; the schema you targeted changed; etc.): try to resolve if obvious. If the resolution would meaningfully change your diff (>20 lines of new logic, or a different public-API choice), STOP and abort: `git rebase --abort`. Then label the PR `bot-blocked-need-decision`, comment with the conflict details, unassign yourself, exit. Don't ship a guessed resolution.
- **Tests that no longer make sense**: if a test you wrote is now testing behavior that doesn't exist anymore (e.g., upstream removed the function you tested), abort + block. Don't delete the test silently.

### Step 4: Re-run tests

The codebase has moved. Your previous "all tests pass" claim is stale. Re-run:

```bash
cd /home/bot/work/meshcore-planner/backend
PYTHONPATH=. /home/bot/work/meshcore-planner/.venv/bin/pytest tests/<module>.py -v
```

For tests requiring deps not in the autobox venv (fastapi, sqlalchemy, etc.), follow the existing protocol from `11-work-phase.md` — acknowledge the gap explicitly in the PR comment, don't claim verification you didn't do.

### Step 5: Re-run pre-PR review (Phase 3)

The diff has changed (different base now). Dispatch a fresh subagent to re-audit per `12-pre-pr-review.md`. The Q1-Q5 verdict in the PR body is now stale; replace it.

If the new verdict is ❌ block → label `bot-blocked-*`, comment, unassign. Don't force-push a verdict-failed branch.

If ⚠ revise → revise. Same 3× rule applies.

If ✅ ship → continue to Step 6.

### Step 6: Force-push with lease

```bash
git push --force-with-lease origin <branch-name>
```

`--force-with-lease` (NOT plain `--force`) refuses the push if anyone else has touched the branch since you fetched it. That protects against the rare race where the reviewer pushed a fixup commit and then asked for rebase — your force-push would otherwise eat their fixup.

### Step 7: Update the PR body + comment

```bash
# Replace the Q1-Q5 audit block with the new subagent verdict
gh pr edit <N> --repo akkerkid/meshcore-planner --body "<new body with refreshed audit>"

# Comment the rebase summary
gh pr comment <N> --repo akkerkid/meshcore-planner --body "<rebase report>"
```

Comment template:

```
Rebased onto upstream/main as of <SHA short>. Summary:

- Conflicts encountered: <none | list with one-line resolution per>
- Tests re-run: <list of test files + pass/fail count>
- Pre-PR Q1-Q5 audit: <new verdict, one-line>
- Force-pushed at <new HEAD short SHA>
```

### Step 8: Remove the `bot-rebase` label

```bash
gh pr edit <N> --repo akkerkid/meshcore-planner --remove-label bot-rebase
```

This signals to the reviewer that the rebase is done. The PR is now ready for re-review.

### Step 9: Update state file

Bump `last_iter_end`, set `last_status: rebased-and-retested`, note the action in the `notes:` array. Then exit the iteration.

## When to abort and block

Abort the rebase + block the PR (`bot-blocked-need-decision`) when:

- A semantic conflict requires a non-obvious design choice
- A test fails after rebase due to upstream changes whose intent you can't infer
- The upstream change deprecated/removed something your PR was building on
- You've rebased + re-rebased twice and still hit conflicts (you're spinning)

The block comment must include:

- What you tried
- What conflicted (with file:line)
- Why you stopped instead of guessing
- What decision you need from AkkerKid (e.g., "should this PR be closed in favor of approach X?")

## What you must NOT do

- Force-push without `--force-with-lease`
- Skip the re-test step
- Skip the re-audit step (Phase 3)
- Edit the PR body's audit block to "still ✅" without actually re-running the subagent
- Resolve conflicts by deleting tests
- Resolve conflicts by reverting upstream's changes
- Apply the `bot-rebase` label yourself — AkkerKid or the reviewer applies it; you only respond to it
