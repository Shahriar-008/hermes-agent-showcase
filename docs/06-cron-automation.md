# 27 Scheduled Cron Jobs — Full Automation Map

> **27 cron jobs across two profiles power everything from personal check-ins to production market data pipelines. This is one of the most automation-dense Hermes deployments documented.**

---

## Overview

| Profile | Jobs | Type |
|---|---|---|
| **default** | 14 | Personal check-ins, reporting, infrastructure maintenance, social/cross-platform |
| **market-channel** | 13 | Market data scanners, reports, research, cross-posting |
| **Total** | **27** | |

---

## Default Profile — 14 Jobs

### Personal Check-Ins

| Job | Schedule (UTC) | Description |
|---|---|---|
| Daily Morning Check-In | 03:00 | Creative daily greeting, varied format |
| Daily Todo Morning Check-In | 09:00 | Check-in with habitrack status, suggest tasks |
| Daily Todo Afternoon Reminder | 14:00 | Afternoon reminder, check habit progress |
| Daily Todo Evening Wrap-Up | 15:00 | Evening wrap-up, habit review, next-day prep |

### Reporting & Monitoring

| Job | Schedule (UTC) | Description |
|---|---|---|
| Daily AI Usage Report | 23:00 | Daily Hermes usage summary via terminal commands |
| Weekly Agent Skills Watch | Mon 04:00 | Checks agentskills.io for new skills matching profile (no_agent script) |
| Delegation Self-Audit | Every 3h | LLM scans DM sessions for delegation violations |

### Infrastructure Maintenance

| Job | Schedule (UTC) | Description |
|---|---|---|
| Hermes GitHub Autobackup | Daily 02:00 | Quick-snapshot backup of `~/.hermes` to GitHub repo |
| Biweekly Gateway Restart | Mon/Thu 04:00 | Restart gateway for memory hygiene (no_agent script) |
| Delegation Watchdog Live | Every 5 min | Python watchdog: checks LCM DB for direct-vs-delegate ratios (no_agent) |

### Social & Cross-Platform

| Job | Schedule (UTC) | Description |
|---|---|---|
| Cutie DM Forwarder | Every minute | Monitors group-chat session DB for Cutie's DMs, forwards to Shahriar |
| X/Twitter Cross-Post (Morning) | 07:00 | Morning tweet: arb alerts, whale moves, market pulse |
| X/Twitter Cross-Post (Evening) | 17:30 | Evening tweet (same logic) |
| X Daily DM Suggestion | 07:30 | Sends ready-to-copy tweet text to DM (manual workflow — no X credits needed) |

---

## Market-Channel Profile — 13 Jobs

All jobs: `no_agent=true`, deliver to the @polymartrend Telegram channel.

### High-Frequency Scanners (P0)

| Job | Schedule | Data Source |
|---|---|---|
| Polytrend Market Alerts ✨ | Every 30 min | CoinGecko, Gamma API, Yahoo Finance, GDELT |
| Polymarket New Markets | Every 15 min | Gamma API — new market listings on Polymarket |
| Polymarket Whale Alerts | Every 15 min | Gamma API — volume >$50K or odds change >10pp |

### Periodic Scanners (P1)

| Job | Schedule | Description |
|---|---|---|
| Crypto Derivatives Scanner | Every 4 hours | BTC/ETH futures, perps, open interest, liquidations |
| ETF Activity Scanner | 13:00, 21:00 | Bitcoin/Ethereum ETF flows, AUM, premiums |
| Cross-Platform Arb v2 (Spredge) | Every 4 hours | 4-platform arbitrage scan (Polymarket, PredictIt, Kalshi, Manifold) |

### Reports & Recaps

| Job | Schedule (UTC) | Description |
|---|---|---|
| Morning Briefing ✨ | 07:00 | Overnight moves, macro events, crypto overview |
| Midday Snapshot ✨ | 13:00 | RSI, MACD, key levels, intraday trends |
| Sentiment Pulse ✨ | 19:00 | Fear & Greed, Polymarket sentiment, headlines |
| Weekly Wrap ✨ | Sun 18:00 | Weekly performance, macro recap, top movers |

### Research (LLM-Enriched)

| Job | Schedule (UTC) | Description |
|---|---|---|
| Macro Catalyst Calendar | Sun 12:00 | Upcoming economic events, earnings, Fed calendar |
| Resolution Watch (Daily) | 14:00 | Markets resolving in next 24-48h, probability trends |
| Resolution Recap (Weekly) | Sun 16:00 | Weekly resolution summary, accuracy stats, big winners |

> ✨ = LLM-enriched via `llm_enricher.py` — raw data piped through deepseek-v4-flash for natural-language commentary.

---

## The Watchdog Pipeline Pattern

Most market-channel cron jobs follow this architecture:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌───────────┐
│ Python Script │ ──▶ │ LLM Enricher │ ──▶ │Shell Wrapper │ ──▶ │ Telegram  │
│ (fetch data)  │     │ (formatting) │     │ (delivery)   │     │ Channel   │
└──────────────┘     └──────────────┘     └──────────────┘     └───────────┘
        │
   ┌────┴────┐
   │ State   │  ← Prevents duplicate posts
   │ File    │
   └─────────┘
```

| Stage | What Happens |
|---|---|
| **1. Fetch** | Python script pulls data from free APIs (CoinGecko, Gamma, Yahoo Finance, GDELT) |
| **2. Enrich** | `llm_enricher.py` pipes raw output through deepseek-v4-flash for natural-language commentary |
| **3. Deliver** | Shell wrapper delivers formatted output to Telegram via the profile bot |
| **4. State-Track** | JSON state files prevent duplicate alerts and maintain cooldowns |

### Silent When Empty

Following the watchdog pattern, all no_agent scripts produce **empty stdout when there's nothing to report**. Empty stdout = silent delivery. Subscribers only see messages when there's actual data.

---

## Error Handling

All market-channel scripts source `_error_handler.sh` — a shared error trap:

```bash
# On any script failure:
# 1. ERR trap fires
# 2. Sends error DM to Shahriar (via @polymarkettrend_bot)
# 3. Exits 0 → cron sees success → nothing delivered to channel
```

This prevents subscriber-facing error spam. Failures are silently routed to the operator's DM, with an append-only `cron_errors.log` as offline backup.

---

## Lessons Learned

1. **Shell `exec` kills error traps.** Every wrapper script originally used `exec python3` which replaces the shell process — ERR traps never fired. Fixed by removing `exec` from all 12 wrappers.
2. **SIGTERM bypasses ERR traps.** The scheduler sends SIGTERM at 120s. Added signal traps (`SIGTERM`, `SIGINT`) to catch timeout kills.
3. **Profile field ≠ delivery routing.** Cron jobs with `profile: X` still deliver via the creating scheduler's adapter. Jobs must be created on the target profile's scheduler.
4. **MACD needs 35+ data points.** Originally Yahoo Finance used `1mo` (~21 trading days) — MACD returned None for all symbols. Fixed by bumping to `3mo` range.
