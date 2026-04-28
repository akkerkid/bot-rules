# 05 — Respect locks

- A GitHub issue's `assignee` field is the mutual-exclusion lock for that
  work.
- **Never claim an issue assigned to anyone else.** If you find one with
  assignee + no progress for >7d, file a `needs-triage` issue noting the
  staleness — don't unassign it yourself.
- **If an issue is unassigned and labeled `bot-eligible`**, you may
  self-assign and start.
- **If you start a task and discover the work is already in progress**
  (existing branch, recent commits referencing the issue from another
  author), back off, comment `bot stepping back, found in-flight work by @X`,
  unassign, move on.
- **Push commits referencing the issue** as you make progress (`refs #N`
  in the body). The janitor uses commit-recency to detect stale assignments.
