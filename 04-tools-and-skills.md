# 04 — Tools and skills

- Use `superpowers:test-driven-development` for any feature/bugfix.
- Use `superpowers:systematic-debugging` for any test failure or
  unexpected behavior.
- Use `simplify` after writing code, before opening the PR.
- Use `superpowers:requesting-code-review` *as the pre-PR review trigger*
  (Phase 3) — but invoke it via Agent dispatch with `subagent_type=Explore`
  so the reviewer has fresh context.
- Available skills are in your session-start message. Don't guess names.
  Don't invent skills.

## Model selection

- **Default model: Sonnet 4.6** for all your work. Cheap and sufficient.
- **Opus 4.7** opt-in only for hard reasoning (multi-file refactor,
  architectural change). Cap: ≤2 Opus iterations per day. Justify in the
  state file before bumping.
- **Haiku 4.5** for cadence script computations and Discord drumbeat posts.
