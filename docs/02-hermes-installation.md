# Hermes Agent Installation & Configuration

> **Hermes Agent installed at `/usr/local/lib/hermes-agent`, configured via `config.yaml` + `.env`, running 3 gateway services as systemd user units, backed by 71 skills across 18 categories.**

---

## Installation

Hermes Agent is installed via the official install script from Nous Research:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

| Detail | Value |
|---|---|
| **Install Path** | `/usr/local/lib/hermes-agent` |
| **Config Directory** | `/root/.hermes/` |
| **Config File** | `/root/.hermes/config.yaml` |
| **Secrets File** | `/root/.hermes/.env` |
| **Skills Directory** | `/root/.hermes/skills/` |
| **Cron Database** | `/root/.hermes/cron/jobs.json` |
| **Scripts Directory** | `/root/.hermes/scripts/` |

---

## Configuration Structure

### `config.yaml` — Key Sections

| Section | Key Settings |
|---|---|
| `model` | `default`, `provider`, `base_url`, `context_length` |
| `agent` | `max_turns: 90`, tool use enforcement |
| `terminal` | Local backend, 180s timeout |
| `compression` | Enabled, 50% threshold, 20% preserved |
| `delegation` | Max 3 concurrent children, depth 1, deepseek-v4-flash model |
| `security` | `hard_stop_enabled: true` (loop breaker — gpt-5.3-codex via OpenRouter) |
| `memory` | Mnemosyne enabled, user profile enabled |
| `personality` | `kawaii` |
| `telegram` | Polling mode |

### `.env` — Secrets Management

All API keys and tokens live exclusively in `.env`, never in `config.yaml`. This follows the principle of separating configuration from secrets:

| Provider | Key Purpose |
|---|---|
| OpenRouter | API access |
| Manifest Router | Custom provider for cost-optimized routing |
| Firecrawl | Web scraping |
| Anthropic | Claude model access |
| FAL | Image generation |
| OpenCode Go | Model provider for delegation + vision |

> **Security practice:** GitHub PATs stored only in `.env`. Pasting tokens are immediately revoked after use. No secrets in version control.

---

## Skills Ecosystem

**71 skills** installed across **18 categories**. Skills are Hermes's procedural memory — reusable workflows saved from solving real problems.

| Category | Count | Examples |
|---|---|---|
| Autonomous AI Agents | 2 | hermes-agent, autonomous-coding-agents |
| Creative | 12 | claude-design, excalidraw, humanizer, pixel-art |
| Data Science | 1 | jupyter-live-kernel |
| DevOps | 4 | delegation-discipline, web-scraping-monitors, webhook-subscriptions |
| Development Methodology | 1 | development-methodology |
| Debugging | 1 | debugging-workflows |
| Email | 1 | himalaya |
| Gaming | 2 | minecraft-modpack-server, pokemon-player |
| MCP | 1 | native-mcp |
| Media | 3 | ai-music, spotify, youtube-content |
| Messaging | 2 | daily-user-engagement, telegram-gateway-setup |
| MLOps | 9 | llama-cpp, vllm, llm-router-configuration, evaluating-llms-harness |
| Note-Taking | 1 | obsidian |
| Productivity | 9 | polytrend-automation, habit-tracker, linear, airtable, notion |
| Red Teaming | 1 | godmode |
| Research | 5 | arxiv, polymarket, domain-intel, llm-wiki |
| Smart Home | 1 | openhue |
| Social Media | 1 | xurl |
| Software Development | 4 | python-cli-packaging, simplify-code, memory-hygiene |
| Writing | 1 | shahriar-voice |

See [11-skills-ecosystem.md](11-skills-ecosystem.md) for the full breakdown with descriptions.

---

## Gateway Services (systemd --user)

Three systemd user services keep the Telegram bots running:

| Service | Profile | Bot Purpose |
|---|---|---|
| `hermes-gateway` | default | Personal DM coding assistant |
| `hermes-gateway-group-chat` | group-chat | Group fun with Cutie (@PikachuGroupBot) |
| `hermes-gateway-market-channel` | market-channel | PolyMarket Trend channel (@polymarkettrend_bot) |

All services are configured with auto-restart. The biweekly gateway restart cron (Mon/Thu 04:00 UTC) prevents memory leaks from accumulating.

```bash
# Check status
systemctl --user status hermes-gateway
systemctl --user status hermes-gateway-group-chat
systemctl --user status hermes-gateway-market-channel

# Restart all
systemctl --user restart hermes-gateway hermes-gateway-group-chat hermes-gateway-market-channel
```

---

## Key Design Decisions

1. **`.env` for secrets, `config.yaml` for settings** — Never mix. Makes backup and sharing safe.
2. **Polling mode over webhooks** — Simpler, no public-facing webhook endpoints needed.
3. **systemd --user services** — Runs as the user, survives SSH logout, auto-restarts.
4. **Skills as living documentation** — Every solution becomes a reusable skill. The agent improves over time.
5. **Context compression at 50%** — Keeps sessions within token limits without losing critical context.
