# brofu plugin

Personal agent skills, packaged as one plugin (`brofu`) for both Claude Code and Codex.

## Skills

| Skill | What it does |
|---|---|
| `lfork` | Fork the current Claude Code or Codex session into a sibling [Luvus](https://luvus.dev) pane with its own resumed session. |

## Install

### Claude Code

```
/plugin marketplace add brofu/plugin
/plugin install brofu@brofu
```

Skills are namespaced by the plugin, e.g. `/brofu:lfork`.

### Codex

```
codex plugin marketplace add brofu/plugin
codex plugin add brofu
```

### Local checkout (no namespace)

Link a skill folder into the agent's skills directory to invoke it as `/lfork` (Claude Code) or `$lfork` (Codex):

```
git clone https://github.com/brofu/plugin.git
ln -s "$PWD/plugin/skills/lfork" ~/.claude/skills/lfork
ln -s "$PWD/plugin/skills/lfork" ~/.agents/skills/lfork
```

## Layout

```
.claude-plugin/          Claude Code plugin + marketplace manifests
.codex-plugin/           Codex plugin manifest
.agents/plugins/         Codex marketplace manifest
skills/<name>/SKILL.md   One folder per skill, shared by both agents
```

## License

MIT
