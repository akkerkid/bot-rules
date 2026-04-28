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
