---
name: lfork
description: Fork the current Claude Code or Codex session into a sibling Luvus pane with its own resumed session. On first use it also installs Luvus's session-id hook for the current agent. Use only when the user explicitly invokes it (/lfork in Claude Code, $lfork in Codex).
argument-hint: "[name for the new Luvus pane/agent]"
disable-model-invocation: true
---

# lfork

Fork the current agent session into a new Luvus pane with `luvus agent fork`. The fork inherits the conversation and gets its own session. Do not use the built-in `/fork` (Claude Code creates a background session with no pane).

The user's argument, when present, is the Luvus name for the new agent (`--name`), used to address it later with `=name` or `luvus agent send name`. It is not the Claude/Codex session name.

## 1. Preflight

Run `echo "$LUVUS_ENV|$LUVUS_BIN_PATH|$LUVUS_PANE_ID"`. If `LUVUS_ENV` is not `1` or either value is empty, this is not a Luvus pane: say so and stop. Use `"$LUVUS_BIN_PATH"` for every Luvus call below.

## 2. Identify the agent and check the session id

Run `"$LUVUS_BIN_PATH" agent get "$LUVUS_PANE_ID"`. Read `.result.agent` (the kind, e.g. `claude` or `codex`) and `.result.session`.

- `session` non-null: go to step 4.
- `session` null: go to step 3.

## 3. Install the session-id hook (one-time setup)

Luvus learns a pane's session id only from a hook that fires when a session starts, so an already-running session stays unknown until it is restarted.

1. Check whether Luvus's hook is already installed, by kind:
   - `claude`: `grep -c luvus ~/.claude/settings.json`
   - `codex`: `grep -c luvus ~/.codex/hooks.json ~/.codex/config.toml`
2. If nothing matches, run `"$LUVUS_BIN_PATH" integration install <kind>`, then show only the lines it added (never print other settings values, they may contain secrets).
3. Either way, tell the user to restart this session in the same pane and run the skill again, then stop:
   - `claude`: exit, then `claude -c`
   - `codex`: exit, then `codex resume --last`

Do not fork now: it would fail with `session_unknown`.

## 4. Fork

Run `"$LUVUS_BIN_PATH" agent fork "$LUVUS_PANE_ID" --no-focus`, adding `--name "<name>"` if one was given. Drop `--no-focus` only if the user asks to jump to the new pane.

On success, report the new pane id and name from the JSON result and stop.

On error, report the code and message verbatim (`unsupported_agent`, `session_unknown`, `spawn_failed`, ...). Never fall back to `pane split`, `agent start`, `agent resume`, or the built-in `/fork`: they do not guarantee an independent copy of the conversation.
