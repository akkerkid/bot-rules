# 12 — Pre-PR review (Phase 3 of each iteration)

Before `gh pr create`, dispatch a fresh-context subagent. Do NOT
inline-review your own diff — your context is biased toward what you wrote.

**This phase is REQUIRED.** A PR opened without a Phase 3 audit block
in its body will be rejected by the reviewer-agent and the bot-discipline
CI check.

**The Q1-Q5 audit must be the OUTPUT of an actual subagent dispatch**,
not text you wrote yourself reciting that you "did the review". The
PR body should include the literal subagent return value, not a paraphrase.

## Dispatch

```
Agent({
  description: "Pre-PR review of bot diff",
  subagent_type: "Explore",
  prompt: <see template below>
})
```

## Prompt template

```
You are reviewing a diff for issue #N. Inputs:
1. The rules: /home/bot/work/bot-rules/*.md
2. The bot's intended diff: `git diff upstream/main...HEAD` (three-dot — what HEAD adds)
3. The actual MERGE diff: `git diff upstream/main..HEAD --stat` (two-dot — what would land if merged)
4. The issue: `gh issue view N`
5. Bot's scratchpad: /home/bot/.bot-state.md

Answer 6 questions, each with file:line citations:

Q1: Does the diff respect every rule? List violations.
Q2: Does the diff duplicate existing code/utilities? Grep for similar
    patterns; cite the existing code that should be extended instead.
Q3: For each new function, trace input parameters 1-2 callers up. What
    units/shape/provenance assumptions are being made? Are they correct?
Q4: For each new function, trace outputs 1-2 callers down. Does the
    consumer want what we produce?
Q5: Are there punted decisions, "TODO"s, or "we should also" comments
    in the diff? Each one needs a follow-up issue OR justification for
    why it's deferred.
Q6: **Branch-base sanity check.** Compare `git diff upstream/main...HEAD --stat`
    (three-dot, what the bot intended) vs `git diff upstream/main..HEAD --stat`
    (two-dot, what the merge would actually do). If the two-dot total is
    materially larger (more files OR significantly more line deletions),
    the branch base is stale and the merge would silently revert work
    that landed on upstream since the branch base. List each file the
    merge would touch but the bot's intended diff doesn't.

    Heuristic: if `two-dot files > 1.5 × three-dot files` OR
    `two-dot deletions > three-dot insertions`, that's a STALE-BASE
    ❌ block — instruct the bot to follow `08-rebase-and-retest.md`.

Verdict: ✅ ship | ⚠ revise (with line-level notes) | ❌ block

Be terse. Cite specifics. Don't rewrite the code — just judge it.
```

## Acting on the verdict

- ✅ ship → `gh pr create`. PR body MUST include:
  - "Closes #N"
  - The Q1-Q5 verdict block VERBATIM from the subagent's response
  - "GPG signed: $(git log -1 --format='%G?')"
  - **Pytest output** (or "tests not runnable on autobox" with rationale per `11-work-phase.md` step 1)
- ⚠ revise → address every note, re-run review.
- 3× ⚠ on the same diff → switch to ❌ (you've stalled).
- ❌ block → label issue `bot-blocked`, post to Discord with the verdict
  text, move on. Don't open the PR.

## Anti-pattern: claiming verification you didn't do

If you find yourself writing phrases like "all tests pass" or
"verified the fix works" without a captured pytest invocation in the
audit block, **STOP**. That's hallucinated verification. Either:

- Run the actual test command (per `11-work-phase.md` step 1) and
  paste the output, OR
- Acknowledge the gap explicitly: `Q1 audit: tests requiring fastapi
  imports were not runnable on autobox; signature is unchanged from
  prior version (verified by diff inspection)`.

This rule is non-negotiable. The reviewer-agent on prod (Plan 4) will
catch hallucinated verification claims and ⚠ the PR.
