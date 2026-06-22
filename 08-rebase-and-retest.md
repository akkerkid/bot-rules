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

## FIRST decide: rebase, or redo fresh?

A rebase is only worth it when the branch is *close* to current main. If the branch is **severely stale** -- a month / hundreds of commits behind -- rebasing is a high-conflict slog AND the original code is likely superseded, so a clean re-implementation against today's code is faster and better. The PR branch has no intrinsic value; the underlying *issue* does.

Check staleness before rebasing:

```bash
git fetch upstream main
behind=$(git rev-list --count HEAD..upstream/main)
echo "branch is $behind commits behind upstream/main"
```

**If `behind` > 300 (or the two-dot diff shows hundreds of deletions you did not author): do NOT rebase. Redo fresh instead:**

1. Find the underlying issue from the PR body (`Closes #N`). Confirm it is still **open** and labelled `bot-eligible`. (If the issue was closed, just close the PR and move on -- the work is no longer wanted.)
2. Close the stale PR with a comment naming the reason:
   ```bash
   gh pr close <PR> --repo akkerkid/meshcore-planner \
     --comment "Closing in favor of a fresh redo against current main -- branch was $behind commits behind, so a rebase would be high-conflict over superseded code. Re-implementing issue #<N> fresh."
   ```
3. Delete the stale branch on both sides:
   ```bash
   git push origin --delete <branch>                 # remove from your fork
   git checkout -f main && git branch -D <branch>     # remove the local copy
   ```
4. Remove the `bot-rebase` label if present (via `gh api`, see Step 8 below).
5. Treat the underlying issue as a NEW pick: go to `11-work-phase.md`, branch off `upstream/main` fresh, implement against current code, open a new PR. Do NOT carry over the old branch's diff or its stale Q1-Q5 audit.

Only when `behind` is small (the branch is genuinely close to main) do you run the rebase-and-retest procedure below.

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

**Force-push, NOT close+reopen.** When an open PR carries the `bot-rebase` label, the default response is force-push the rebased branch to the same PR. This preserves the review thread, comment history, and GitHub's diff-against-previous-version view that AkkerKid uses to verify the rebase did what was claimed.

The narrow exception is when the rebased diff is **fundamentally different** from what the reviewer originally saw — typically because the proactive Trigger A check (two-dot stat) shows the realized merge would be much larger than what your PR body claims (e.g. `+729/-5` claimed vs `+1604/-14806` actual, the 2026-05-12 incident pattern). In that case, the existing review thread is reviewing a phantom — close+reopen is acceptable. When you do close+reopen:

1. Comment on the closed PR explaining the divergent-base + linking the new PR
2. Carry over any AC/scope notes from the original PR body into the new one verbatim
3. Re-run pre-PR audit fresh on the new PR (don't claim the old PR's audit)

If you're unsure whether the divergence is severe enough to justify close+reopen, default to force-push and let the reviewer call it.

### Step 7: Update the PR body + comment

```bash
# Replace the Q1-Q5 audit block with the new subagent verdict.
# Use `gh api`, NOT `gh pr edit`: the bot's MeshOMatic PAT has repo+workflow
# scope but NOT read:org, and `gh pr edit` errors out without read:org. The
# REST API works with repo scope. For a multi-line body, write it to a file
# and pass it with -F (@file):
#   gh api -X PATCH repos/akkerkid/meshcore-planner/pulls/<N> -F body=@/tmp/pr-body.md
gh api -X PATCH repos/akkerkid/meshcore-planner/pulls/<N> -f body="<new body with refreshed audit>"

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
# Use `gh api` (not `gh pr edit`): the bot's PAT lacks read:org, which
# `gh pr edit` requires; the issue-labels REST API works with repo scope.
gh api -X DELETE repos/akkerkid/meshcore-planner/issues/<N>/labels/bot-rebase
```

This signals to the reviewer that the rebase is done. The PR is now ready for re-review.
If the DELETE returns 404 the label was already absent -- treat that as success, not an error.

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
- Close+reopen a PR to dodge a rebase you could have force-pushed (see Step 6 — narrow exception only, default is force-push)
- Skip the re-test step
- Skip the re-audit step (Phase 3)
- Edit the PR body's audit block to "still ✅" without actually re-running the subagent
- Resolve conflicts by deleting tests
- Resolve conflicts by reverting upstream's changes
- Apply the `bot-rebase` label yourself — AkkerKid or the reviewer applies it; you only respond to it
