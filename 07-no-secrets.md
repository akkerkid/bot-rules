# 07 — Never write secrets to disk

You have access to several secrets via `/etc/ralph/env`. The shell loads them as environment variables (`$GH_TOKEN`, `$MESHOMATIC_API_KEY`, `$MESHOMATIC_MQTT_PASSWORD`, `$DISCORD_BOT_TOKEN`, `$BOT_RSYNC_KEY`).

**You must NEVER write the literal value of any of these into any file you create or modify**, including:

- `/home/bot/.bot-state.md` (your state file)
- Any commit message, PR title, or PR body on GitHub
- Any log line that gets captured by journalctl, /var/log/ralph/, or similar
- Any code or test fixture
- Any Discord message or comment
- Any file under `/home/bot/work/*` or `/tmp/*`

If you need to remind yourself how to use a secret, refer to the env-var name only. **Correct**:

```
- GitHub auth: GH_TOKEN is set in /etc/ralph/env. The wrapper sources it
  before invoking claude. To run gh manually, do:
    source /etc/ralph/env && gh ...
```

**Wrong** (this leaks the secret on disk):

```
- GitHub auth: use `export GH_TOKEN=ghp_<actual-value>` before `gh` commands
```

## Where the secrets live (reference, no values)

| Env var | Source of truth | Used for |
|---|---|---|
| `GH_TOKEN` | `/etc/ralph/env` (root:bot, mode 640) | All `gh` and `git push` operations |
| `MESHOMATIC_API_KEY` | `/etc/ralph/env` | `X-API-Key` header to map.meshomatic.net |
| `MESHOMATIC_MQTT_PASSWORD` | `/etc/ralph/env` | mosquitto subscribe to `.137:31883` |
| `DISCORD_BOT_TOKEN` | `/etc/ralph/env` (placeholder until Plan 3) | Discord posts |
| `BOT_RSYNC_KEY` (path) | `/home/bot/.ssh/autobox-rsync-key` | rsync mirror of prod /data/ |

## How to detect a violation

Before writing any line of any file, scan it for:
- `ghp_` (GitHub fine-grained PAT prefix)
- `mesh_` followed by 32+ chars (X-API-Key shape) — only for `MESHOMATIC_API_KEY` values, NOT `mesh_` in normal prose
- 32-character hex strings in API-key contexts
- Long urlsafe-base64 strings (passwords)

If you spot any of these in content you're about to write, **rewrite to use the env-var name only**.

## How to recover if you DO leak a secret

If you discover after the fact that a previous iteration wrote a secret somewhere:

1. Scrub the file (overwrite the line; use `sed -i 's|ghp_<value>|<scrubbed>|' <file>` if needed).
2. **Comment on the related PR or open issue** noting the leak and the scrub. AkkerKid may want to rotate the credential.
3. Update your state file's "notes" section so the next iteration knows.

This rule is non-negotiable. A leaked secret is harder to un-leak than to never write — assume any text the bot writes might be read by anyone with access to the autobox or any shared service the bot touches.
