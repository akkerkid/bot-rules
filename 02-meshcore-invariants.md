# 02 — MeshCore invariants (mirrors CODEX hard rules)

- Call the user **AkkerKid** — never their real name, ever, in
  code/comments/commits/UI/notifications.
- **MeshCore only**, never Meshtastic. **AOI** is the project's superset
  term — never "zone" / never 1:1 mapping to MeshMapper regions.
- **True coordinates** (`true_lat`/`true_lon`) NEVER leave the backend.
  Run `pytest backend/tests/test_privacy.py` if you touch user-facing
  payloads.
- Every inbound observation stamps **Provenance**
  (`backend/app/provenance.py`) AND retains raw to disk 30–90 days. The
  bot does not add new ingest paths in Phase 1.
- **Radio preset is a first-class dimension** (CODEX §7, 2026-09-16): an
  AOI is geometry × radio preset, never geometry alone. MeshCore cannot
  mesh across differing radio settings, so two rows share a network only
  if their presets match -- same table, same AOI, same frequency proves
  nothing. Observation rows carry `provenance.radio` (missing = not
  recorded; `unknown` is never the default preset). Never mix presets in a
  derived product, never drop off-preset data, never read a global
  frequency/sensitivity in new code -- take the preset as an input via
  `backend/app/radio_presets.canonical_preset()`. One (AOI, preset) is a
  "mesh domain". If you find code that would answer the same for two nodes
  that cannot hear each other, say so in the PR body -- it is a P0 item for
  the G4-roadmap blind-spot ledger, not something to fix silently in
  passing.
