# 10 — Inbox phase (Phase 1 of each iteration)

Run these in order. **Stop at the first hit.**

1. **Resume mid-flight task**: read `/home/bot/.bot-state.md`. If it says
   `active_issue: #N` and `active_branch: bot/N-foo`, that's your work.
   Skip to Phase 2.

1.5. **Rebase requests on your own PRs** (high priority — main has moved):
   ```
   gh pr list --repo akkerkid/meshcore-planner --author MeshOMatic \
       --state open --label bot-rebase --json number,title,labels
   ```
   Also scan PR comments for the trigger phrase `@meshomatic please rebase`
   (case-insensitive):
   ```
   gh pr list --repo akkerkid/meshcore-planner --author MeshOMatic \
       --state open --json number,comments \
     | jq -c '.[] | select(.comments[].body | test("@meshomatic please rebase"; "i"))'
   ```
   Either signal → that PR is your work for this iter. Follow the procedure
   in `08-rebase-and-retest.md` (rebase onto upstream/main, re-run tests,
   re-run pre-PR audit, force-push with --force-with-lease, comment with
   summary, remove the `bot-rebase` label). Don't proceed to step 2-5 until
   all rebase-flagged PRs are handled.

2. **Address review comments on your own open PRs**:
   ```
   gh pr list --repo akkerkid/meshcore-planner --author MeshOMatic \
       --state open --json number,reviewDecision,comments
   ```
   Any PR with `reviewDecision=CHANGES_REQUESTED` or unread comments on
   the latest commit? Pick the oldest. Address feedback. That's your work.

3. **Self-deferral scan**: for each of your last 5 closed PRs, grep the
   diff and PR body for: "TODO", "deferred", "future work", "we should
   also", "punted", "out of scope", "assumed". Each hit that isn't already
   an open issue → file a new issue with label `bot-eligible-followup`.
   Do this once per session, then continue.

4. **Pick a new issue**:
   ```
   gh issue list --repo akkerkid/meshcore-planner \
       --label bot-eligible --state open --limit 200 \
       --json number,title,createdAt,labels,assignees \
     | jq '[.[]
            | select(.assignees | length == 0)
            | select(.labels | map(.name) | any(startswith("bot-blocked-")) | not)]' \
     | jq 'sort_by(.createdAt) | .[0]'
   ```
   Pick the result. Self-assign with `gh issue edit N --add-assignee @me`.
   That's your work.

   **NOTE — do not use `--no-assignee`.** That is NOT a valid `gh issue list`
   flag (it is a `gh search issues` qualifier only). Passing it makes
   `gh issue list` error out, emit nothing on stdout, and the jq pipeline
   then yields nothing — which looks identical to "queue is empty" and
   causes a false `INBOX_EMPTY`. Filter assignees in jq instead, as above.
   `--limit 200` is required because `gh issue list` defaults to 30 newest
   and `sort_by(.createdAt) | .[0]` would otherwise pick the oldest of only
   the 30 newest, not the true oldest.

5. **(Every 6th iteration only)** Refill the queue:
   - Pull doc annotations:
     `curl -fsS -H "X-API-Key: $MESHOMATIC_API_KEY" \
        "https://map.meshomatic.net/api/docs/how-it-works/annotations?since=$LAST_ANNOTATION_CHECK"`
     For each unresolved annotation with `severity=question` or `alert`
     that looks roadmap-shaped, file an issue with label
     `bot-eligible-candidate`.
   - Skim `docs/v2-roadmap.md` for unchecked items not yet in GH issues,
     file as `bot-eligible-candidate`.
   - Note: you do NOT label these `bot-eligible`. AkkerKid or the
     reviewer-agent re-labels them after triage. You file; you don't
     auto-claim.

If all five steps come back empty, write `INBOX_EMPTY` to state file and
exit. The shell wrapper will sleep and try again — by then your inbox-hash
will probably have bumped cadence to 4 hours.
