# Self-Monitoring, Watchdog & Guardrail System

> **A multi-layered self-monitoring stack — Python watchdog checking tool call ratios every 5 minutes, LLM-driven self-audit every 3 hours, and a hard loop breaker — evolved through 3 iterations from enforcement to behavior shaping.**

---

## The Three Layers

```
┌─────────────────────────────────────────────┐
│              GUARDRAIL STACK                  │
│                                              │
│  ┌─────────────────────────────────────┐    │
│  │  Layer 3: Loop Breaker              │    │
│  │  hard_stop_enabled=true             │    │
│  │  gpt-5.3-codex via OpenRouter       │    │
│  │  Prevents infinite agent loops      │    │
│  └─────────────────────────────────────┘    │
│                   ▲                          │
│  ┌─────────────────────────────────────┐    │
│  │  Layer 2: Self-Audit (every 3h)     │    │
│  │  LLM scans DM sessions              │    │
│  │  CRITICAL/WARNING/INFO tiers        │    │
│  │  Reports delegation violations      │    │
│  └─────────────────────────────────────┘    │
│                   ▲                          │
│  ┌─────────────────────────────────────┐    │
│  │  Layer 1: Watchdog (every 5 min)    │    │
│  │  Python script reads LCM DB         │    │
│  │  Direct:Delegate ratio threshold    │    │
│  │  Telegram alert + stop_signal.json  │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

---

## Layer 1: Delegation Watchdog

| Detail | Value |
|---|---|
| **Script** | `delegation_watchdog.py` |
| **Schedule** | Every 5 minutes (no_agent script) |
| **Method** | Reads LCM database, counts direct tool calls vs `delegate_task` calls per session |
| **Threshold** | >5:1 direct-to-delegate ratio |
| **Actions** | Writes `stop_signal.json` → Telegram alert → Logs violation |

### How It Works

```python
# Pseudocode of the watchdog logic
for session in recent_sessions():
    direct_calls = count_direct_tool_calls(session)
    delegate_calls = count_delegate_calls(session)
    
    if delegate_calls == 0:
        ratio = float('inf')
    else:
        ratio = direct_calls / delegate_calls
    
    if ratio > 5.0:
        alert(f"Session {session.id}: {direct_calls} direct vs {delegate_calls} delegate (ratio {ratio:.1f}:1)")
```

### Track Record

The watchdog has successfully caught violations multiple times. It's been code-reviewed and improved by Claude Code — fixed 4 missing Mnemosyne tools, typed exceptions, and variable naming. Previously patched 8+ times during guardrail experiments.

---

## Layer 2: Delegation Self-Audit

| Detail | Value |
|---|---|
| **Schedule** | Every 3 hours (LLM-driven) |
| **Method** | Uses `session_search` to scan DM sessions, counts tool calls, compares ratios |
| **Output** | Structured report with CRITICAL/WARNING/INFO tiers delivered to Shahriar via DM |
| **Feedback** | Attempts to save learning notes to Mnemosyne memory |

### Report Tiers

| Tier | Trigger | Action |
|---|---|---|
| **CRITICAL** | Session with zero delegations despite multi-step work | Immediate alert |
| **WARNING** | Ratio above threshold but borderline | Noted for review |
| **INFO** | Healthy delegation patterns | Confirmation only |

---

## Layer 3: Loop Breaker

| Detail | Value |
|---|---|
| **Config** | `hard_stop_enabled: true` |
| **Model** | gpt-5.3-codex via OpenRouter |
| **Purpose** | Prevents infinite agent loops — if the agent keeps making tool calls without converging, the loop breaker stops it |

This is a last-resort safety net, not an active monitoring tool. It rarely triggers because the watchdog and audit layers catch issues earlier.

---

## Guardrail Evolution — 3 Approaches Tried

### Attempt 1: Reasoning Degrade (June 3) ❌

**Idea:** When the watchdog hits RED tier, reduce the agent's `reasoning_effort`. Less reasoning = faster/cheaper, and theoretically the agent would delegate more.

**Why it failed:** Created a death spiral. The degraded agent made MORE delegation mistakes, which triggered MORE watchdog alerts, which degraded reasoning further. A negative feedback loop.

### Attempt 2: Hard Guardrail Block (June 3) ❌

**Idea:** A `tool_guardrails.py` patch that blocked direct tool calls past a threshold. The agent literally couldn't call tools directly — it HAD to delegate.

**Why it failed:** The guardrail blocked itself. When the guardrail code needed to run tool calls, it was blocked by its own rules. Required the user to manually say "delegate this" to unblock. Too aggressive — it broke the agent's ability to function.

### Attempt 3: SOUL.md + Watchdog (Current) ✅

**Approach:** Self-discipline via SOUL.md pre-task gate rules + watchdog alerts as correction signals.

| Component | Role |
|---|---|
| **SOUL.md** | Contains delegation rules in "Technical Workflow" section. The agent reads this every turn as part of its system prompt. |
| **Watchdog** | External monitor (not self-enforcing). Alerts the user when patterns slip. The user corrects; the agent learns. |
| **Audit** | Retrospective analysis. Shows patterns over time, not real-time enforcement. |

**The philosophy:** Shahriar explicitly chose natural behavior shaping over brute-force enforcement. The agent learns through feedback, not punishment.

> *"I'd rather the agent get better over time than be locked in a cage."*

---

## Why Enforcement Was Rejected

| Enforcement | Behavior Shaping |
|---|---|
| Blocks tool calls | Alerts user to patterns |
| Can block itself | External — can't self-sabotage |
| Degrades under pressure | Stable regardless of state |
| User must manually override | User sees report and adjusts |
| Feels like fighting the tool | Feels like improving the tool |

The current approach treats the agent like an engineer who needs feedback, not a process that needs a kill switch.

---

## Lessons Learned

1. **Enforcement that can block itself is worse than no enforcement.** The hard guardrail was a single point of failure.
2. **Negative feedback loops are real.** Degrading reasoning → worse decisions → more degradation. Always consider second-order effects.
3. **External monitoring beats internal enforcement.** The watchdog runs as a separate process (no_agent cron). It can't be affected by the agent's state.
4. **Cron debugging requires full pipeline trace.** Exit → trap → scheduler → deliver. Don't claim a fix until every stage is verified.
5. **The watchdog has been patched 8+ times.** Self-monitoring is iterative — you discover edge cases, fix them, discover more.
