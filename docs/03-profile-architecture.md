# Multi-Profile Architecture & Security Isolation

> **Three fully isolated Hermes profiles — personal coding partner, group-chat bot, and market channel automation — each with its own config, skills, cron, memory, and Telegram bot identity.**

---

## The Three Profiles

| Profile | Role | Primary Model | Memory | Bot |
|---|---|---|---|---|
| **default** | Personal coding/DevOps partner + orchestrator | deepseek-v4-pro (OpenCode Go) | Mnemosyne ON | @hermespikachu_bot |
| **group-chat** | Fun with Cutie (Rayta) in "Shahriar, Cutie and Pikachu" group | deepseek-v4-flash (OpenRouter) | Mnemosyne OFF | @PikachuGroupBot |
| **market-channel** | PolyMarket Trend channel automation & delivery | N/A (script-only cron) | N/A | @polymarkettrend_bot |

---

## Why Three Profiles?

A single Hermes installation can run multiple independent profiles. Each profile is a separate agent identity with its own:

- **Personality & system prompt** — The coding partner is technical and direct; the group bot is playful; the market bot has no personality (script-only)
- **Model assignment** — Different models optimized for different use cases
- **Skills** — Only relevant skills are active per profile
- **Memory** — Work knowledge never leaks into group conversations
- **Cron jobs** — Separate automation schedules
- **Telegram bot** — Different bot tokens = different identities in Telegram

---

## Security Isolation Matrix

```
┌──────────────────────────────────────────────────────────────┐
│                    ISOLATION BOUNDARIES                       │
│                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────┐   │
│  │  default    │   │  group-chat  │   │  market-channel  │   │
│  │             │   │              │   │                  │   │
│  │ skills/    │   │ skills/     │   │ scripts/        │   │
│  │ cron/      │   │ cron/       │   │ cron/           │   │
│  │ memories/  │   │ memories/   │   │ .env            │   │
│  │ .env       │   │ .env        │   │ (own bot token) │   │
│  └─────────────┘   └──────────────┘   └──────────────────┘   │
│         │                 │                    │              │
│    ─────┼─────────────────┼────────────────────┼───────       │
│         │         NO CROSS-PROFILE DATA SHARING              │
│    ─────┼─────────────────┼────────────────────┼───────       │
│         ▼                 ▼                    ▼              │
│   @hermespikachu   @PikachuGroupBot    @polymarkettrend_bot   │
└──────────────────────────────────────────────────────────────┘
```

### Group-Chat: Maximum Lockdown

The group-chat profile has **26 toolsets disabled** — the most restricted of all profiles:

| Decision | Rationale |
|---|---|
| Memory OFF | Group conversations must never be persisted or searched later |
| No delegation | Group bot doesn't spawn subagents — it's a chat companion |
| No terminal | Can't execute shell commands from the group |
| No file tools | Can't read/write files from the group |
| No cronjob | Can't schedule work from the group |
| No web tools | Can't browse or search the web from the group |

> **The rule:** Work knowledge stays in the default profile's private DMs. The group bot is purely for social interaction.

### Market-Channel: Script-Only

The market-channel profile runs zero LLM-driven conversations. All 13 cron jobs use `no_agent=true` — Python scripts fetch data, format output, and deliver directly to Telegram. The LLM enricher (`llm_enricher.py`) is called programmatically from within scripts, not as part of an agent conversation loop.

---

## Profile Directory Layout

Each profile mirrors the root `~/.hermes/` structure:

```
~/.hermes/profiles/market-channel/
├── .env                  # Own bot token, API keys
├── config.yaml           # Profile-specific config
├── SOUL.md              # Profile personality
├── skills/              # Profile-specific skills
├── scripts/             # 30+ Python/bash scripts
├── cron/                # jobs.json (13 jobs)
├── logs/                # agent.log, cron_errors.log
├── sessions/            # Session transcripts
└── output/              # Cron job output archives
```

---

## Lessons Learned

1. **Profile isolation is not automatic.** Cron jobs must be created on the target profile's scheduler, not the root scheduler. The `profile` field only sets the script execution environment — delivery always goes through the scheduler's Telegram adapter.
2. **`.env` is per-profile.** A profile's `.env` does not inherit from the root `.env`. API keys must be copied explicitly.
3. **Memory isolation requires explicit config.** Without setting `mnemosyne.data_dir` per profile, profiles silently share the default Mnemosyne database.
4. **Cross-profile cron delivery is a pitfall.** Jobs on the root scheduler deliver via the root bot regardless of the `profile` field. For market-channel delivery via @polymarkettrend_bot, the jobs MUST live on the market-channel profile's own scheduler.
5. **Group-chat has 26 toolsets disabled for a reason.** The group bot is purely for social interaction with Cutie. Work knowledge (terminal access, file tools, web search, delegation, cron jobs) must never leak into social conversations. The group-chat profile is locked down to prevent any accidental work-mode leakage — the bot can only chat, nothing else.
