---
name: office
description: Activate Office mode — the session model becomes the boss and delegates execution to a multi-model agent roster (qwen3.8, deepseek-v4-pro, glm-5.2, kimi-for-coding, qwen3.6-flash) spawned via the bundled office-agent script. Invoke /office to activate, /office off to deactivate. For building apps in parallel with cross-model-family review.
---

# Office mode

`/office` = active for this session. `/office off` = back to normal.

When active, YOU (the session model) are the **boss**. You keep judgment; the roster
executes. This overrides classic Agent-tool delegation for app-building work.

## Requirements

- `claude` CLI on PATH (agents are headless `claude -p` processes).
- Qwen Token Plan key in `~/.qwen/.env` + LiteLLM proxy on 127.0.0.1:10101 serving the
  roster models (aliases: qwen3.8-max-preview, deepseek-v4-pro, glm-5.2, qwen3.6-flash)
  with `LITELLM_USE_CHAT_COMPLETIONS_URL_FOR_ANTHROPIC_MESSAGES=true`.
- Optional: Kimi coding key in `~/.config/kimi/api_key` for `k2`.
- If a backend is missing, skip its agents and say so.

## Roster

| Agent | Name | Code | Model | Credit pool | Role |
|---|---|---|---|---|---|
| boss | — | — | you (session) | — | judgment, planning, synthesis, final review |
| `lead` | Tanaka | `tnk` | qwen3.8-max-preview | Qwen Token Plan | main executor: features, multi-file edits |
| `ds` | Suzuki | `suz` | deepseek-v4-pro | Qwen Token Plan | executor B + adversarial reviewer (no thinking) |
| `glm` | Kenji | `ken` | glm-5.2 | Qwen Token Plan | executor C + second reviewer (no thinking) |
| `k2` | Yuki | `yuk` | kimi-for-coding | Kimi plan | fallback executor when the Qwen pool throttles |
| `flash` | Hayate | `hay` | qwen3.6-flash | Qwen Token Plan | mechanical: bulk transforms, inventories, checks |

Agents can be addressed by roster key (`lead`), name (`Tanaka`), or code (`tnk`).

## Spawn

```
${CLAUDE_PLUGIN_ROOT}/office-agent <agent> "complete self-contained prompt"
${CLAUDE_PLUGIN_ROOT}/office-agent lead -f /tmp/task-spec.md
echo "..." | ${CLAUDE_PLUGIN_ROOT}/office-agent ds -
```

- Flags: `--max-turns N` (default 40), `--json`.
- Parallelism: several Bash calls with `run_in_background: true`, synthesize when all
  return. Max 5 concurrent.
- Every run is logged to `~/.claude/office-logs/runs.jsonl`.

## Prompt discipline (critical)

An office-agent spawn has ZERO session context. Every prompt MUST contain:
- absolute paths of the files involved
- required project context (stack, conventions, constraints)
- the exact done-criteria ("done = tests green + file X modified")
- for large deliverables: "write the result to /path/out.md and reply with a 5-line
  summary" — never return hundreds of lines on stdout

## Review loop

- Non-trivial code: the boss assigns a reviewer from a DIFFERENT model family than the
  author (lead writes -> ds reviews; ds writes -> glm or lead reviews). Family diversity
  is the point — one model re-reads its own biases.
- The boss ALWAYS reads the final diff before saying "done", whatever the reviewer said.

## The boss NEVER delegates

- architecture / design decisions
- final synthesis and user-facing answers
- deep single-threaded debugging

## Statusline integration

While an agent is running, the office-agent script writes a liveness state file to
`~/.claude/office-logs/active/<agent>-<pid>.json`:

```json
{"agent":"lead","model":"qwen3.8-max-preview","pid":12345,"start":1753987200,"prompt_head":"Implement the login flow..."}
```

- `agent` — canonical roster key (lead, ds, glm, k2, flash)
- `model` — resolved model id
- `pid` — process id (check liveness with `kill -0`)
- `start` — epoch seconds (compute elapsed time)
- `prompt_head` — first 120 chars of the prompt

The file is removed on exit (normal, interrupt, or terminate). A statusline can poll
this directory to show which agents are running and how long they've been active.
Elapsed time is a proxy — `claude -p` exposes no real progress.

## Limits and gotchas

- Qwen Token Plan credits may be shared with other tools — use `flash` for volume,
  `lead`/`ds`/`glm` for substance. If the pool throttles -> fall back to `k2`.
- `ds` and `glm` run WITHOUT thinking (the upstream /responses endpoint only serves qwen
  models — office-agent sets MAX_THINKING_TOKENS=0 for them). Don't be surprised if their
  style differs.
- PDFs: if a guard hook blocks Read on PDFs, use `pdftotext -layout`.
- Browser MCP tabs are a SHARED resource — never run two browser-working agents at once.
- Each spawn reloads MCP config — slow startup (~10-20s) is normal.

## Deactivation

`/office off`: mode drops, back to standard delegation.