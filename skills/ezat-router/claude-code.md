# Claude Code on EZAT Router

Claude Code needs two values. Nothing else is required: Claude Code's own model names are aliases on EZAT Router.

| Variable | Value | Sent as |
| --- | --- | --- |
| `ANTHROPIC_BASE_URL` | `https://api.router.ezat.io` (no `/v1`) | |
| `ANTHROPIC_AUTH_TOKEN` | the EZAT Router key | `Authorization: Bearer` |

`ANTHROPIC_API_KEY` (sent as `x-api-key`) also works, but interactive Claude Code asks the user once to approve it. Prefer `ANTHROPIC_AUTH_TOKEN`.

## Steps

1. Check the key first (`quickstart.md`, steps 1–3).
2. Ask the user how they want it set up:
   - **This terminal only:**

     ```bash
     export ANTHROPIC_BASE_URL="https://api.router.ezat.io"
     export ANTHROPIC_AUTH_TOKEN="$EZAT_API_KEY"
     claude
     ```

   - **Every terminal:** add the two `export` lines to the shell profile (`~/.zshrc` or `~/.bashrc`), after the line that sets `EZAT_API_KEY`.
   - **Everywhere Claude Code runs, including the IDE extensions' spawned sessions and background agents:** the `env` block of `~/.claude/settings.json`. Merge, don't overwrite, and don't print the key. For example:

     ```bash
     python3 - <<'EOF'
     import json, os, pathlib
     path = pathlib.Path.home() / ".claude" / "settings.json"
     settings = json.loads(path.read_text()) if path.exists() else {}
     env = settings.setdefault("env", {})
     env["ANTHROPIC_BASE_URL"] = "https://api.router.ezat.io"
     env["ANTHROPIC_AUTH_TOKEN"] = os.environ["EZAT_API_KEY"]
     path.parent.mkdir(parents=True, exist_ok=True)
     path.write_text(json.dumps(settings, indent=2) + "\n")
     print("updated", path)
     EOF
     ```

     For one project only, use that project's `.claude/settings.local.json`, and make sure git ignores it.
3. Never put the key in a project's `.claude/settings.json`: it is committed and shared.
4. Verify: start `claude`, run `/status`. The `Anthropic base URL` line must show `https://api.router.ezat.io`, and an `Auth token` line must name `ANTHROPIC_AUTH_TOKEN`. Then send a short prompt; it should answer, and the request shows on `https://router.ezat.io/usage`.

## Precedence and conflicts

- A value in a settings file's `env` block wins over a shell export. If `/status` shows the wrong base URL, look in `~/.claude/settings.json`, `.claude/settings.json` and `.claude/settings.local.json`.
- A saved claude.ai login plus a token gives a startup warning about two credentials. The token is used; `/logout` removes the warning.
- An old `ANTHROPIC_API_KEY` (for example an Anthropic key) set next to `ANTHROPIC_AUTH_TOKEN` is sent as `x-api-key`, which EZAT Router reads first: `401 Invalid API key`. Unset it, or set it to the EZAT Router key too.
- An `ANTHROPIC_API_KEY` that was declined once is ignored silently; re-enable it in `/config` (**Use custom API key**) or switch to `ANTHROPIC_AUTH_TOKEN`.
- The VS Code extension reads its own setting, `claudeCode.environmentVariables`, for its login check. Put both variables there for the extension.

## Problems

| Symptom | Fix |
| --- | --- |
| `404` naming `/v1/v1/messages` | Remove `/v1` from `ANTHROPIC_BASE_URL` |
| `401` | See above; check the key with `quickstart.md` |
| `402` | No credit: the user adds it at `https://router.ezat.io/billing` |
| `404` `model: …` | The model name isn't served. Set `ANTHROPIC_MODEL` to an id from `GET https://api.router.ezat.io/v1/models` |
| `400` naming unknown fields or beta features | Set `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1` in the same place as the other variables |
| Claude Code asks to log in | The token didn't reach it: set it in the shell or in `~/.claude/settings.json` |

Full page: `https://docs.router.ezat.io/tools/claude-code.md`.
