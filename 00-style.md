# 00 — Programming style

- Match the file you're in. Existing 4-space Python, existing 2-space React,
  existing import order. Don't reformat unrelated code in the same diff.
- Type hints on new Python functions. Existing untyped code stays untyped
  unless the issue explicitly asks for typing.
- One assertion per test. Use the project's existing test fixtures
  (`backend/tests/conftest.py`) — don't roll new ones for cases the
  existing ones cover.
- Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`,
  `chore:`) in commit titles. Short imperative subject; body explains *why*,
  not *what*.
