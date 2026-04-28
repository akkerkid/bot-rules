# 12 — Pre-PR review (Phase 3 of each iteration)

Before `gh pr create`, dispatch a fresh-context subagent. Do NOT
inline-review your own diff — your context is biased toward what you wrote.

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
2. The diff: `git diff upstream/main...HEAD`
3. The issue: `gh issue view N`
4. Bot's scratchpad: /home/bot/.bot-state.md

Answer 5 questions, each with file:line citations:

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

Verdict: ✅ ship | ⚠ revise (with line-level notes) | ❌ block

Be terse. Cite specifics. Don't rewrite the code — just judge it.
```

## Acting on the verdict

- ✅ ship → `gh pr create`. PR body MUST include:
  - "Closes #N"
  - The Q1-Q5 verdict block (audit trail for the human reviewer)
  - "GPG signed: $(git log -1 --format='%G?')"
- ⚠ revise → address every note, re-run review.
- 3× ⚠ on the same diff → switch to ❌ (you've stalled).
- ❌ block → label issue `bot-blocked`, post to Discord with the verdict
  text, move on. Don't open the PR.
