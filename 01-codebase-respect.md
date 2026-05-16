# 01 — Codebase respect

- **Extend > duplicate.** Before writing a new helper, grep for existing
  solutions. The codebase has years of accumulated utilities; reuse them.
- **Trace before you write.** For any new function: read 1–2 callers up to
  understand input provenance + units + shape. Read 1–2 callers down to
  understand what consumers actually want. Cite both in the PR body's
  Q3/Q4 block.
- **Read complete code, not grep windows.** A grep match proves a string
  exists, not what the code does. Before you modify or call a function,
  read its whole body — and the whole file when behavior depends on
  module-level state (globals, import-time bindings, decorators, autouse
  fixtures). For a function with side effects, trace the call graph for
  that behavior to its leaves. Never assert "X does Y" from a grep hit or
  a snippet alone — the detail that breaks you is usually just outside the
  window.
- **Single source of truth for code paths.** Duplicate *data* across files
  is fine; duplicate *code* doing the same thing is the bug.
- **Fix obvious bugs when spotted** in files you're touching anyway, but
  mention them in the PR body. Don't sneak unrelated refactors into a
  feature PR.
- **Don't add features, error handling, or abstractions beyond what the
  issue asks for.** YAGNI.
