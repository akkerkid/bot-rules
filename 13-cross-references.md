# 13 — Code↔doc cross-references

Every code file and every `.md` doc that names code paths must carry
matching `DOCS:` markers, in the same PR. Touch-driven — you only need to
maintain refs for files your diff touches.

## When you touch a code file

Before staging:

```bash
cd /home/bot/work/meshcore-planner
grep -rln '<file-you-touched>' docs/ host/ meshcore-memory/ \
  ../meshomatic-conduit-firmware/docs/ ../meshomatic-flagship/docs/ \
  2>/dev/null
```

For every match: ensure the touched file carries a matching `DOCS:` line.
If no doc names the file, do nothing — don't add placeholders.

## When you touch a `.md` and add a code-file path in prose

In the same diff, add a `DOCS: <doc-path-from-repo-root>` line inside the
referenced code file's docstring.

## Where the `DOCS:` line goes

**INSIDE the function or file docstring**, not above the signature.
Agents (you included) grep for function names and read forward; lines
above the `def` are invisible.

Python — module docstring at top of file:

```python
"""Unified TX dispatcher and pacer.

DOCS: docs/how-it-works/D7-virtual-companion-system.md
DOCS: meshcore-memory/project_unified_tx_phase_a.md
"""
```

Python — function docstring:

```python
def _publish_packet(self, ...):
    """Single canonical TX entry point.

    DOCS: docs/how-it-works/D7-virtual-companion-system.md#tx-dispatcher
    """
```

JavaScript — JSDoc:

```javascript
/**
 * App routing and layout.
 *
 * DOCS: docs/how-it-works/G1-frontend-architecture.md
 */
```

Bash — first line **inside** function body:

```bash
reload_backend() {
    # DOCS: host/DEBUG.md#service-restart-recipes
    ...
}
```

C/C++ — first line inside `{`:

```cpp
void MQTTBridge::resetSlot0AllTripped() {
    // DOCS: docs/conduit_broker_override.md#all-tripped-recovery
    ...
}
```

## Path format

- **Code-side `DOCS:` paths**: repo-rooted (no leading `/`, no `../..`).
  Example: `DOCS: docs/how-it-works/D7-virtual-companion-system.md`.
- **Doc-side link targets** (in `.md` files): file-relative for in-repo
  refs. Example from `host/CODEX.md`: `[D7](../docs/how-it-works/D7-...md)`.
- **Cross-repo or AkkerKid-host-only paths** (`/root/.claude/...`,
  sibling repos): absolute, with `(AkkerKid's host only)` annotation in
  the description.

## Format rules

- One ref per `DOCS:` line.
- Optional `— short description` after the path, for human-skimming.
- Anchor links (`#section`) encouraged for large docs.
- **Append, don't reorder.** New entries at the end of the existing
  `DOCS:` block. Reordering is a merge-conflict generator and other
  agents may have their own additions in flight.
- Removals only when the doc itself is deleted or renamed (in the same
  PR that does the rename).

## Verification

`host/docs-health.sh` runs forward + reverse checks (advisory):

- **Forward**: every `DOCS:` line in code resolves to an existing `.md`.
- **Reverse**: coverage stat — how many code files mentioned in docs
  carry `DOCS:` markers.

Run before PR if your diff touched many code+doc files together. Forward
warnings on your diff are blockers — fix them.

## Pre-PR audit (Q1 — see `12-pre-pr-review.md`)

The Q1 audit subagent should check this rule. If your diff:

1. Touches code AND a doc names that code without a matching `DOCS:`
   line — Q1 violation.
2. Adds a code-file path in a `.md` without a matching `DOCS:` line in
   the code — Q1 violation.
3. Renames or deletes a doc without updating every `DOCS:` line that
   pointed at it — Q1 violation.

## Edge cases

- Tests for `foo.py` go in `tests/test_foo.py`. The `DOCS:` line goes
  in the production code, not the test. Test files don't need their own
  `DOCS:` block unless a doc explicitly discusses the testing approach.
- Generated code, migrations, one-off scripts, `__init__.py`, barrel
  files: skip. They get rewritten or aren't load-bearing.
- Illustrative refs in docs (tutorial code with fake paths): skip.
  These get added to the docs-health reverse-check allowlist if they
  cause noise.

## Why

Bidirectional refs let humans and agents answer "what context exists
for this code?" without grep-fishing five doc trees. Touch-driven scope
means refs accumulate naturally where work happens.

## Full convention

`/home/bot/work/meshcore-planner/host/notes/feedback_cross_references.md`
— per-language examples, merge-conflict story, full edge-case table.
Read it once, refer back when the format question is non-obvious.
