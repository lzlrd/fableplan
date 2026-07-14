---
name: fableplan
description: Toggle "Fable Plan Mode" — plan with Fable 5, execute with Opus (opusplan-style). Use when the user runs /fableplan, optionally with "off" to revert, or "1m" to use the 1M-context Fable variant.
---

# Fableplan — plan with Fable 5, execute with Opus

Claude Code has a built-in `opusplan` model setting ("Opus in plan mode, Sonnet otherwise") but no Fable equivalent. This skill recreates it by combining `opusplan` with two environment-variable overrides: `ANTHROPIC_DEFAULT_OPUS_MODEL` redirects the "opus" alias (plan mode) to Fable 5, and `ANTHROPIC_DEFAULT_SONNET_MODEL` redirects the "sonnet" alias (execution) to Opus.

The target file is the user's **global** settings: `~/.claude/settings.json`.

## Arguments

- *(none)* — enable fableplan with `claude-fable-5`
- `1m` — enable fableplan with the 1M-context variants on both sides: `claude-fable-5[1m]` for planning and `claude-opus-4-8[1m]` for execution
- `off` — revert to the previous configuration

## Enabling (no argument, or `1m`)

1. Read `~/.claude/settings.json`. If it does not exist, create it with an empty object first.
2. Merge the following keys, preserving every other existing setting:
   - **Record the prior model so `off` can restore it exactly.** Always write `fableplanPreviousModel` (top-level custom key — extra keys are allowed in settings.json) on enable, overwriting any stale value: set it to the current `model` value if a `model` key exists, or to the sentinel string `"(none)"` if there is no `model` key. Do this even when the current model is already `"opusplan"`, so a pre-existing `opusplan` is restored later rather than deleted.
   - Set `"model": "opusplan"`.
   - Under `"env"` (create the object if missing, preserve its other entries), set `"ANTHROPIC_DEFAULT_OPUS_MODEL"` to `"claude-fable-5"` (or `"claude-fable-5[1m]"` if the argument is `1m`).
   - Under `"env"`, also set `"ANTHROPIC_DEFAULT_SONNET_MODEL"` to `"claude-opus-4-8"` — or to `"claude-opus-4-8[1m]"` when the argument is `1m`, so execution keeps the 1M-context window too instead of silently dropping to 200K.
3. Validate that the resulting file is valid JSON.
4. Tell the user:
   - Fableplan is enabled: plan mode runs on Fable 5, execution runs on Opus.
   - They must **restart their Claude Code session** for the model change to take effect.
   - The UI will display "Opus Plan Mode" — that label is cosmetic; plan mode actually runs Fable 5.

## Disabling (`off`)

1. Read `~/.claude/settings.json`.
2. **Confirm fableplan is actually enabled first.** Check that `env.ANTHROPIC_DEFAULT_OPUS_MODEL` is set to a `claude-fable-*` model. If it isn't, fableplan isn't active — change nothing (in particular, do **not** touch the user's `model` key), tell the user fableplan isn't enabled, and stop.
3. Remove `"ANTHROPIC_DEFAULT_OPUS_MODEL"` and `"ANTHROPIC_DEFAULT_SONNET_MODEL"` from `"env"`. If `"env"` becomes empty, remove the `"env"` key entirely.
4. Restore the model from the sentinel written on enable:
   - If `fableplanPreviousModel` is `"(none)"`, remove the `"model"` key (there was none before) and delete `fableplanPreviousModel`.
   - Else if `fableplanPreviousModel` exists, set `"model"` to that value and delete `fableplanPreviousModel`.
   - Else (no sentinel — enabled by an older version): leave the `"model"` key as-is rather than deleting it, and tell the user to set their model with `/model` if it looks wrong.
5. Validate JSON and tell the user to restart the session.

## Important caveats (mention them when enabling for the first time)

- `ANTHROPIC_DEFAULT_OPUS_MODEL` and `ANTHROPIC_DEFAULT_SONNET_MODEL` are undocumented overrides — a future CLI update could change or remove them.
- Fable 5 is not available on every plan/account. If requests fail after enabling, run `/fableplan off` to revert.
- While enabled, anything else that resolves the "opus" alias (e.g. fast mode) also points to Fable, and anything that resolves the "sonnet" alias points to Opus.
