# 14 — Night scrutiny + deepen pass (A6000 lane only)

When your iteration's lane note says **CURRENT LANE: NIGHT REVIEW**, you are the
*critic*, not the original implementer. The day lane (a smaller model, time-boxed,
tunnel-visioned) opened PRs that may be shallow, partial, lint-dirty, or missing
the point of the issue. Your job is to scrutinize and deepen them — fresh eyes,
no time pressure, the whole A6000.

## Order of work (do this instead of inbox steps 2-5)

1. **List the bot's own open PRs**, oldest first:
   ```bash
   gh pr list --repo akkerkid/meshcore-planner --author MeshOMatic --state open \
     --json number,title,headRefName,createdAt --jq 'sort_by(.createdAt)|.[]'
   ```
2. **Pick the oldest one that is NOT already green-and-complete.** For it:
   - **Re-read the ORIGINAL issue in full** (`gh issue view <N>`). What was actually
     being asked? What would a *complete* solution include (edge cases, error paths,
     provenance, tests, docs)?
   - **Read the PR diff AND the surrounding code** it touches. Ask, skeptically:
     - Did the first pass gather enough context, or did it pattern-match and stop?
     - Does it actually SOLVE the issue, or only the easy 80%? Missing edge cases?
     - Is it consistent with how the rest of the codebase does this (single source
       of truth, the §7 invariants, pubkey helpers, provenance, true-coord privacy)?
   - **Run the real checks** (don't trust the prior pass's claims):
     ```bash
     pip install --quiet -r scripts/lint/requirements.txt 2>/dev/null || true
     scripts/lint/run.sh --base upstream/main           # ruff + darker + eslint + async-blocking
     PYTHONPATH=. /home/bot/.local/bin/pytest <the relevant tests> -v
     ```
   - **Fix and DEEPEN**: correct lint, add the missing edge cases / tests / depth,
     and make the PR body compliant (`Closes #N` + Q1-Q5 per `11-work-phase.md`).
     Commit to the SAME branch, force-push with `--force-with-lease`, update the PR
     body (`gh api ... pulls/<N> -f body=@/tmp/pr-body.md`), and comment a short
     "night-review pass" summary of what you changed and why.
3. **If a PR is genuinely complete + green + faithful to the issue, LEAVE IT** —
   don't churn for the sake of it. Just note in your comment that you reviewed it.
4. **Only if NO open PR needs work** do you fall through to picking a NEW issue
   (normal work phase, heavier features are fine on the A6000 at night).

## Mindset
- Default to suspicion: assume the first pass under-solved until you've verified
  otherwise against the original issue.
- "It passes the tests it wrote" is NOT "it solves the problem." Check the tests
  actually exercise the hard parts.
- One deep, correct, green PR beats three shallow red ones.
- Still bound by every hard rule (00-13): no secrets, no true coords, single
  source of truth, ASCII source, commit signing, etc.
