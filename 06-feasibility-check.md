# 06 — Feasibility check (run as Work-phase step 0)

After picking up an issue, do a 5-minute pre-flight before writing any
code. Ask three questions explicitly:

1. **Data**: does this issue require data my autobox doesn't have a mirror
   of? Examples: UTS tiles for an AOI not in `/data/uts-tiles/`, MQTT topics
   that prod's broker doesn't publish, PostGIS tables not in the sanitized
   dump.
   - Check: `ls /data/uts-tiles/`, `mosquitto_sub -h 172.30.30.137 -t '<topic>' -W 5`,
     `psql -d bot_sanitized -c '\dt'`.

2. **Hardware**: does this issue require physical hardware I don't have?
   Examples: USB radio, conduit access, mesh-DM round-trip, GPU.
   - Phase 1: NO USB radio. Anything firmware/RF is out.

3. **Decision**: does this issue require a product/business decision?
   Examples: "should we add AOI X", "deprecate Y", "redesign Z UI".
   - If the issue doesn't have a clear engineering answer in the body or
     in linked docs, escalate.

If any answer is "no, I can't" → stop, comment, label, unassign, move on:

  Missing data:      label `bot-blocked-need-data`
  Missing hardware:  label `bot-blocked-need-hardware`
  Needs a decision:  label `bot-blocked-need-decision`

Comment template:
  > Pre-flight blocked. Reason: <data|hardware|decision>.
  > Specifically missing: <what>.
  > To unblock: <smallest concrete unblock action>.
  > Bot stepping back. Re-add label `bot-eligible` (and remove
  > `bot-blocked-*`) to re-queue.

Then post a one-line summary to #meshomatic-bot via say-on-discord and
unassign yourself.

## Pattern detection

If you've blocked 3+ issues in the last week with the same
`Specifically missing:` reason, file ONE `bot-meta-request` issue
describing the pattern: e.g. "I've been blocked on UTS tiles for AOI=KGSO
4 times this week. Could that AOI be added to my mirror?" One meta-issue
per pattern, not one per blocked issue.
