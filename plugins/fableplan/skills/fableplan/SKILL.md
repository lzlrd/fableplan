---
name: fableplan
description: Toggle "Fable Plan Mode" — plan with Fable 5, execute with Opus (opusplan-style). Use when the user runs /fableplan, optionally with "off" to revert, or "1m" to use the 1M-context Fable variant.
---

# Fableplan — plan with Fable 5, execute with Opus

Claude Code has a built-in `opusplan` model setting ("Opus in plan mode, Sonnet otherwise") but no Fable equivalent. This skill recreates it by combining `opusplan` with two environment-variable overrides: `ANTHROPIC_DEFAULT_OPUS_MODEL` redirects the "opus" alias (plan mode) to Fable 5, and `ANTHROPIC_DEFAULT_SONNET_MODEL` redirects the "sonnet" alias (execution) to Opus.

The target file is the user's **global** settings: `~/.claude/settings.json`.

## Arguments

- *(none)* — enable fableplan with `claude-fable-5`
- `1m` — enable fableplan with `claude-fable-5[1m]` (1M context window)
- `off` — revert to the previous configuration

## Enabling (no argument, or `1m`)

1. Read `~/.claude/settings.json`. If it does not exist, create it with an empty object first.
2. Merge the following keys, preserving every other existing setting:
   - If a `model` key already exists and is different from `"opusplan"`, save its current value under the key `fableplanPreviousModel` (top-level, custom key — extra keys are allowed in settings.json) so it can be restored later.
   - Set `"model": "opusplan"`.
   - Under `"env"` (create the object if missing, preserve its other entries), set `"ANTHROPIC_DEFAULT_OPUS_MODEL"` to `"claude-fable-5"` (or `"claude-fable-5[1m]"` if the argument is `1m`).
   - Under `"env"`, also set `"ANTHROPIC_DEFAULT_SONNET_MODEL"` to `"claude-opus-4-8"` so execution runs on Opus instead of Sonnet.
3. Validate that the resulting file is valid JSON.
4. Tell the user:
   - Fableplan is enabled: plan mode runs on Fable 5, execution runs on Opus.
   - They must **restart their Claude Code session** for the model change to take effect.
   - The UI will display "Opus Plan Mode" — that label is cosmetic; plan mode actually runs Fable 5.

## Disabling (`off`)

1. Read `~/.claude/settings.json`.
2. Remove `"ANTHROPIC_DEFAULT_OPUS_MODEL"` and `"ANTHROPIC_DEFAULT_SONNET_MODEL"` from `"env"`. If `"env"` becomes empty, remove the `"env"` key entirely.
3. Restore the model:
   - If `fableplanPreviousModel` exists, set `"model"` to that value and remove the `fableplanPreviousModel` key.
   - Otherwise remove the `"model"` key (falls back to the account default) and tell the user they can pick a model with `/model`.
4. Validate JSON and tell the user to restart the session.

## Important caveats (mention them when enabling for the first time)

- `ANTHROPIC_DEFAULT_OPUS_MODEL` and `ANTHROPIC_DEFAULT_SONNET_MODEL` are undocumented overrides — a future CLI update could change or remove them.
- Fable 5 is not available on every plan/account. If requests fail after enabling, run `/fableplan off` to revert.
- While enabled, anything else that resolves the "opus" alias (e.g. fast mode) also points to Fable, and anything that resolves the "sonnet" alias points to Opus.
