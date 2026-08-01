# claude-office

![Office mode — pixel art cover](assets/office.gif)

**Office mode for Claude Code**: your session model stays the **boss** (judgment, planning, final review) and delegates execution to a roster of headless agents running under *other* models — with cross-model-family review, so a model never grades its own biases.

## The roster

| Agent | Name | Code | Model | Role |
|---|---|---|---|---|
| boss | — | — | you (session model) | judgment, planning, synthesis, final review |
| `lead` | Tanaka | `tnk` | qwen3.8-max-preview | main executor: features, multi-file edits |
| `ds` | Suzuki | `suz` | deepseek-v4-flash-0731 | executor B + adversarial reviewer |
| `glm` | Kenji | `ken` | glm-5.2 | executor C + second reviewer |
| `k2` | Yuki | `yuk` | kimi-for-coding | fallback executor (separate credit pool) |
| `flash` | Hayate | `hay` | qwen3.6-flash | mechanical bulk work |

## Usage

```
/office                       # activate for the session
/office off                   # deactivate
```

When active, the session model spawns roster agents via the bundled `office-agent` script (headless `claude -p` routed through a local LiteLLM proxy or the Kimi endpoint), fans work out in parallel, and runs cross-family review (author != reviewer) before merging.

## Getting started

**You don't need this exact roster.** Any OpenAI-compatible provider (Alibaba MaaS,
DeepSeek, OpenRouter, Together, local Ollama...) works through the bundled LiteLLM
setup, and the roster is just a table at the top of `office-agent` you edit to match.

Full walkthrough: **[SETUP.md](SETUP.md)** — install, proxy config
([examples/litellm-office.yaml](examples/litellm-office.yaml)), keeping it alive
([examples/com.office.litellm.plist](examples/com.office.litellm.plist)), pointing the
roster at your models, statusline integration
([examples/statusline-office.sh](examples/statusline-office.sh)) and troubleshooting.

Quick version:

```bash
git clone https://github.com/Wecko-ai/claude-office.git /tmp/claude-office
cp -r /tmp/claude-office ~/.claude/skills/office
cp /tmp/claude-office/office-agent /tmp/claude-office/office-watch ~/bin/
# configure the proxy (SETUP.md step 2), then:
office-agent lead --task "smoke test" "reply with just: pong"
```

Then `/office` in any Claude Code session.

## Live office view

Two ways to watch the agents work:

- **`office-hq`** — localhost dashboard (auto-started by `/office`, or `office-hq --daemon`):
  http://localhost:4545 shows every desk with task + real-time action, sessions grouped,
  and the recent runs feed. Zero dependencies, stdlib only. `office-hq --stop` kills it.
- **`office-watch`** — same thing as a terminal TUI for a split pane.

## Requirements

- `claude` CLI on PATH (agents are headless `claude -p` processes) + `jq`, `zsh`, `python3`
- An API key for at least one OpenAI-compatible provider + `pip install litellm`
- Optional: Kimi coding key in `~/.config/kimi/api_key` for `k2`
- If a backend is missing, its agents are skipped and the boss says so

Full operational manual: [SKILL.md](SKILL.md).

## Install (as plugin)

The skill follows the Claude Code plugin layout (`.claude-plugin/plugin.json`), so it can also be installed from any marketplace that references this repo.

## License

MIT — [Wecko](https://wecko.ai)
