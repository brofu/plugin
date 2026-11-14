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

## 2. Resolve the current session

Determine the invoking agent from the current conversation (Codex or Claude), not from the pane returned by Luvus. In Codex, `LUVUS_PANE_ID` can be inherited from an unrelated pane through a shared daemon. Never fork or install another agent's hook merely because that pane reports a different kind.

For Codex, use the current `CODEX_SESSION_ID` (or `CODEX_THREAD_ID`) and `"$LUVUS_BIN_PATH" agent list` to find the Codex agent whose `session` equals that ID. Use its `pane` as the target only if exactly one matches. If the current session ID is missing or multiple panes match, explain the problem and stop; do not guess from focus, cwd, or the inherited pane ID.

If no Codex pane matches, go to step 3 for Codex. For Claude, query `"$LUVUS_BIN_PATH" agent get "$LUVUS_PANE_ID"`; require `.result.agent` to be `claude`. If its `session` is non-null, use that pane as the target; if null, go to step 3 for Claude. On a kind mismatch, stop and explain the identity mismatch.

## 3. Install the session-id hook (one-time setup)

Luvus needs a session binding before it can fork. The Codex integration handles shared daemon environments and can report a session on a subsequent prompt; Claude reports session identity through its session hook.

1. Check whether Luvus's hook is already installed, by kind:
   - `claude`: `grep -c luvus ~/.claude/settings.json`
   - `codex`: `grep -c luvus ~/.codex/hooks.json ~/.codex/config.toml`
2. If nothing matches, run `"$LUVUS_BIN_PATH" integration install <kind>`, then show only the lines it added (never print other settings values, they may contain secrets).
3. For Codex, stop and tell the user to invoke the skill again in the same pane so the prompt hook can report the session. If it still has no binding, restart in the same pane and resume the exact current session with `codex resume <session-id>`, then invoke the skill again. Do not substitute `--last` for a known session ID. For Claude, stop and tell the user to exit, run `claude -c` in the same pane, and invoke the skill again.

Do not fork now: it would fail with `session_unknown`.

## 4. Fork

Run `"$LUVUS_BIN_PATH" agent fork <resolved-pane> --no-focus`, adding `--name "<name>"` if one was given. Drop `--no-focus` only if the user asks to jump to the new pane.

On success, report the new pane id and name from the JSON result and stop.

On error, report the code and message verbatim (`unsupported_agent`, `session_unknown`, `spawn_failed`, ...). Never fall back to `pane split`, `agent start`, `agent resume`, or the built-in `/fork`: they do not guarantee an independent copy of the conversation.
