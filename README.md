# fableplan

**Plan with Fable 5, execute with Opus** — an [opusplan](https://docs.claude.com/en/docs/claude-code)-style dual-model setup for Claude Code.

Claude Code ships with a built-in `opusplan` model setting: *"Opus in plan mode, Sonnet otherwise"*. Fable is selectable in the model picker on its own, but there is no `fableplan` counterpart — no built-in way to run Fable in plan mode and something else for execution. This plugin recreates it with a small trick: `opusplan` resolves its plan-mode model through the `ANTHROPIC_DEFAULT_OPUS_MODEL` environment variable and its execution model through `ANTHROPIC_DEFAULT_SONNET_MODEL`, so pointing the first at Fable 5 and the second at Opus gives you **Fable 5 in plan mode, Opus for execution**.

## Install

```
/plugin marketplace add lzlrd/marketplace
/plugin install fableplan@lzlrd
```

## Usage

| Command | Effect |
|---|---|
| `/fableplan` | Enable: plan with `claude-fable-5`, execute with Opus 5 (`claude-opus-5`) |
| `/fableplan 1m` | Same, 1M-context on both sides: plan `claude-fable-5[1m]`, execute `claude-opus-5[1m]` |
| `/fableplan off` | Revert to your previous model configuration |

Restart your Claude Code session after toggling — the model setting is read at startup.

## How it works

The skill merges these keys into your global `~/.claude/settings.json` (everything else is preserved):

```json
{
  "model": "opusplan",
  "env": {
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-fable-5",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "claude-opus-5"
  }
}
```

That's it. You can also apply this by hand without installing anything.

## Caveats

- The UI will display **"Opus Plan Mode"** — the label is cosmetic; plan mode actually runs Fable 5. After restarting, confirm with `/status` rather than trusting the label.
- `ANTHROPIC_DEFAULT_OPUS_MODEL` and `ANTHROPIC_DEFAULT_SONNET_MODEL` are undocumented overrides and could change in a future CLI release. `ANTHROPIC_DEFAULT_FABLE_MODEL` exists too, but it only renames the Fable entry — it can't give you a plan-mode split.
- **Managed or policy settings can ignore model overrides** (`ignoreModelOverrides`, an `availableModels` allowlist, a model-access entitlement). On a restricted account planning silently falls back to the newest permitted Opus with no error and no change to the label.
- Fable 5 is not available on every plan/account. If requests fail, run `/fableplan off`.
- While enabled, anything else that resolves the "opus" alias (e.g. fast mode) also points to Fable, and anything resolving the "sonnet" alias points to Opus.

## License

MIT
