# PolyMarket Trend — Automated Market Intelligence Channel

> **@polymartrend — a Telegram channel delivering automated prediction market analysis via 13 cron jobs, powered by a custom Python bot (827 lines), 4-platform arbitrage scanning, and LLM-enriched reports — all from free public APIs.**

---

## Channel Overview

| Detail | Value |
|---|---|
| **Channel** | [@polymartrend](https://t.me/polymartrend) |
| **Brand** | "Spredge" (for arbitrage) / "PolyTrend" (for reports) |
| **Bot** | @polymarkettrend_bot |
| **Profile** | market-channel (fully isolated) |
| **Updates** | 5-10 automated posts daily |
| **Data Sources** | 6+ free public APIs (zero paid keys) |

---

## Architecture

```
┌───────────────────────────────────────────────────────┐
│              market-channel profile                    │
│                                                       │
│  ┌──────────────────────┐   ┌──────────────────────┐  │
│  │   marketbot.py       │   │  llm_enricher.py     │  │
│  │   (827 lines)        │──▶│  (deepseek-v4-flash) │  │
│  │   Core data fetcher  │   │  Natural-language     │  │
│  └──────────────────────┘   └──────────────────────┘  │
│            │                          │                │
│  ┌─────────┼──────────────┐          │                │
│  │         ▼              ▼          ▼                │
│  │  ┌──────────┐  ┌────────────┐  ┌───────────┐     │
│  │  │ Scanners │  │ Arb Engine │  │  Reports  │     │
│  │  │ (3 jobs) │  │ (1 job)    │  │ (4 jobs)  │     │
│  │  └──────────┘  └────────────┘  └───────────┘     │
│  └───────────────────────────────────────────────────┤
│                                                       │
│  13 cron jobs → @polymarkettrend_bot → @polymartrend  │
└───────────────────────────────────────────────────────┘
```

---

## Core Bot: marketbot.py

The heart of the automation — 827 lines of pure Python 3:

| Feature | Description |
|---|---|
| **Alert Mode** | Price moves >8%, volume spikes >2x, Polymarket odds changes >8%, GDELT headlines |
| **Morning Briefing** | Overnight market moves, macro events, crypto overview |
| **Midday Snapshot** | RSI(14), MACD(12,26,9) for 11 symbols (SPY, QQQ, NVDA, BTC, ETH, gold, oil...) |
| **Sentiment Pulse** | Fear & Greed Index, Polymarket sentiment, news headlines |
| **Weekly Wrap** | Weekly performance recap, macro summary, next week preview |
| **Dependencies** | Zero external packages — Python 3 stdlib + `requests` only |

---

## Spredge — 4-Platform Arbitrage Scanner

Schedule: Every 4 hours. Brand: "Spredge" (Spread + Edge).

| Platform | Type | Matching |
|---|---|---|
| **Polymarket** | Real-money | Primary — top 60 binary markets |
| **PredictIt** | Real-money | Binary-only (2-contract markets) |
| **Kalshi** | Real-money | Events API, category-filtered |
| **Manifold** | Play-money | Bridge platform — tags markets with `[Polymarket]` origin |

### Matching Engine (Composite v2.2)

A 4-factor scoring system replaces simple Jaccard matching:

| Factor | Weight | Description |
|---|---|---|
| Raw Jaccard | Base | Token overlap on stopword-stripped text |
| Entity-Normalized Jaccard | Boost | Canonical entity names (US→unitedstates, Iran→iran, etc.) |
| Date Bonus | +0.08/match | Matching date fragments (month+day, year) |
| Keyword Boost | +0.04/match | Signal keywords (nuclear, peace, deal, election, tariff, etc.) |

Final score = max(raw_jac, entity_jac) + date_bonus + kw_boost. Threshold: 0.20.

### Track Record

The scanner maintains an `arb_history.db` SQLite database tracking every opportunity:

| Metric | Tracking |
|---|---|
| **Total Tracked** | 80+ opportunities |
| **Win Rate** | Calculated from resolved arbs only (not total tracked) |
| **Executable** | Polymarket ↔ PredictIt / Kalshi (real-money) |
| **Signal** | Polymarket ↔ Manifold (play-money → directional signal only) |
| **Paper Portfolio** | Virtual $1,000 with auto-tracking of every executable trade |

---

## LLM Enrichment Layer

All 5 report modes pipe through `llm_enricher.py`:

```bash
marketbot.py [mode] | llm_enricher.py --mode [mode]
```

| Mode | Enrichment | Prompt |
|---|---|---|
| `watch` | 📋 Context | 1-sentence explanation per alert block |
| `morning` | 📋 Market Narrative | 2-3 sentences connecting movers, news, Polymarket |
| `midday` | 📋 Technical Context | Interpret RSI/MACD collectively |
| `sentiment` | 📋 Sentiment Breakdown | Connect Fear & Greed, headlines, Polymarket pulse |
| `weekly` | 📋 Week in Review | Summarize the week's narrative |

**API:** OpenCode Go endpoint, `deepseek-v4-flash`, `max_tokens: 1024` (critical — lower values cause empty responses from reasoning models).

**Graceful degradation:** If the LLM API call fails after 1 retry, the enricher passes raw marketbot output through unchanged. Subscribers never see error messages.

---

## Data Sources (All Free)

| Source | Data | API |
|---|---|---|
| **CoinGecko** | Crypto prices (BTC, ETH, top altcoins) | Free, no key |
| **Binance Futures** | Funding rates, open interest, liquidations | Free, no key |
| **Yahoo Finance** | Stocks, ETFs, commodities | Free, no key |
| **Gamma API** | Polymarket markets, events, odds, volume | Free, no key |
| **GDELT Project** | News headlines (macro/geopolitical filters) | Free, no key |
| **Alternative.me** | Fear & Greed Index | Free, no key |

> **Zero paid API keys.** All data comes from free public endpoints. This keeps the channel cost-free to operate.

---

## X/Twitter Cross-Posting

| Detail | Value |
|---|---|
| **CLI** | xurl (OAuth 2.0 PKCE) |
| **Account** | @ShahJoy23 |
| **Schedule** | 07:00 and 17:30 UTC daily |
| **Content** | Best arb, whale move, or market pulse |
| **Status** | CreditsDepleted — automated posting blocked |
| **Fallback** | DM suggestion at 07:30 UTC — ready-to-copy tweet text sent to Telegram DM for manual posting |

---

## State Management

| File | Purpose |
|---|---|
| `marketbot_state.json` | Alert cooldowns, seen news headlines |
| `poly_whale_alerts_state.json` | Odds snapshots, 4h cooldown per market |
| `poly_new_markets_state.json` | Seen market IDs (pruned to last 5000) |
| `cross_arb_v2_state.json` | Arb cooldowns (8h per matched pair) |
| `arb_history.db` | SQLite arb opportunity tracker with resolution tracking |
| `crypto_derivatives_state.json` | Funding/OI snapshots for derivative signals |
| `etf_activity_state.json` | ETF volume/divergence state with cooldowns |
| `x_crosspost_state.json` | Posted arb/whale/pulse IDs for dedup |

---

## Resolution Watch — Full Lifecycle Tracking

A 3-phase system for tracking market outcomes:

| Phase | Script | What It Does |
|---|---|---|
| **Scanner** | `resolution_scanner.py` | Finds markets closing within 7 days, 5-factor dispute risk scoring, category-tagged |
| **Tracker** | `resolution_tracker.py` | Tracks reported markets, checks resolution outcomes, calculates accuracy % |
| **Monitor** | `dispute_monitor.py` | Detects UMA-pending resolutions, recently closed markets, anomaly detection |

The recap (`resolution_recapper.py`) runs all three and produces a weekly accountability report with accuracy stats, dispute watch, and new closing markets.

---

## Lessons Learned

1. **Win rate must divide by resolved, not total.** Original formula showed 0% with 80 tracked and 0 resolved. Fixed: display "N/A (none resolved)" until markets close.
2. **Paper portfolio shouldn't show unrealized losses.** Unsettled trades are "at risk," not "lost." Separate unsettled from realized P&L.
3. **Kalshi rate-limiting kills the arb pipeline.** Without retry logic, a single 429 lost the entire executable bridge. Added exponential backoff with 2 retries.
4. **`max_tokens: 500` returns empty from reasoning models.** DeepSeek consumes tokens for internal reasoning before producing output. Minimum 1024.
5. **Shell `exec` prevents error traps — systemic across all wrappers.** Every script needed `exec` removed to enable the shared error handler.
6. **X/Twitter automated posting is blocked (CreditsDepleted), but the DM fallback keeps content flowing.** When xurl hit the API credit limit, automated posting to @ShahJoy23 stopped. The workaround: a cron job at 07:30 UTC sends a ready-to-copy tweet (bare code block, zero wrapper) to Shahriar's Telegram DM. One tap-hold-copy-paste on mobile — minimal friction, maximum reliability. The channel doesn't depend on X for distribution; Telegram is the primary platform.
