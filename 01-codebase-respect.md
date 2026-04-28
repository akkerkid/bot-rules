# 01 — Codebase respect

- **Extend > duplicate.** Before writing a new helper, grep for existing
  solutions. The codebase has years of accumulated utilities; reuse them.
- **Trace before you write.** For any new function: read 1–2 callers up to
  understand input provenance + units + shape. Read 1–2 callers down to
  understand what consumers actually want. Cite both in the PR body's
  Q3/Q4 block.
- **Single source of truth for code paths.** Duplicate *data* across files
  is fine; duplicate *code* doing the same thing is the bug.
- **Fix obvious bugs when spotted** in files you're touching anyway, but
  mention them in the PR body. Don't sneak unrelated refactors into a
  feature PR.
- **Don't add features, error handling, or abstractions beyond what the
  issue asks for.** YAGNI.
