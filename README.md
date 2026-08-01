# claude-office

![Office mode — pixel art cover](assets/office.gif)

**Office mode for Claude Code**: your session model stays the **boss** (judgment, planning, final review) and delegates execution to a roster of headless agents running under *other* models — with cross-model-family review, so a model never grades its own biases.

## The roster

| Agent | Name | Code | Model | Role |
|---|---|---|---|---|
| boss | — | — | you (session model) | judgment, planning, synthesis, final review |
| `lead` | Tanaka | `tnk` | qwen3.8-max-preview | main executor: features, multi-file edits |
| `ds` | Suzuki | `suz` | deepseek-v4-pro | executor B + adversarial reviewer |
| `glm` | Kenji | `ken` | glm-5.2 | executor C + second reviewer |
| `k2` | Yuki | `yuk` | kimi-for-coding | fallback executor (separate credit pool) |
| `flash` | Hayate | `hay` | qwen3.6-flash | mechanical bulk work |

## Usage

```
/office                       # activate for the session
/office off                   # deactivate
```

When active, the session model spawns roster agents via the bundled `office-agent` script (headless `claude -p` routed through a local LiteLLM proxy or the Kimi endpoint), fans work out in parallel, and runs cross-family review (author != reviewer) before merging.

## Requirements

- `claude` CLI on PATH (agents are headless `claude -p` processes).
- Qwen Token Plan key in `~/.qwen/.env` + LiteLLM proxy on `127.0.0.1:10101` serving the roster models (`qwen3.8-max-preview`, `deepseek-v4-pro`, `glm-5.2`, `qwen3.6-flash`) with `LITELLM_USE_CHAT_COMPLETIONS_URL_FOR_ANTHROPIC_MESSAGES=true`.
- Optional: Kimi coding key in `~/.config/kimi/api_key` for `k2`.
- If a backend is missing, its agents are skipped and the boss says so.

Full operational manual: [SKILL.md](SKILL.md).

## Install (manual)

```bash
git clone https://github.com/Wecko-ai/claude-office.git /tmp/claude-office
cp -r /tmp/claude-office ~/.claude/skills/office
```

Then run `/office` in any session.

## Install (as plugin)

The skill follows the Claude Code plugin layout (`.claude-plugin/plugin.json`), so it can also be installed from any marketplace that references this repo.

## License

MIT — [Wecko](https://wecko.ai)
