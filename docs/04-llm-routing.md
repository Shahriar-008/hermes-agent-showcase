# 5-Tier LLM Router — Cost-Optimized Model Selection

> **The Manifest Router automatically selects from 5 model tiers based on task complexity — free models for simple queries, paid reasoning models only when the task demands it.**

---

## The Manifest Router

The Manifest Router (`app.manifest.build`) is configured as a custom provider in Hermes Agent. It intercepts every LLM request and routes it to the appropriate model tier:

```
User Message
     │
     ▼
┌─────────────────┐
│ Manifest Router │  ← "How complex is this task?"
└────────┬────────┘
         │
    ┌────┼────┬────┬────┐
    ▼    ▼    ▼    ▼    ▼
 Simple Std  Cmplx Reas Fallback
```

---

## The 5 Tiers

| Tier | Model | Provider | Cost/M Tokens | Use Case |
|---|---|---|---|---|
| **Simple** | GPT-5.4-mini | Copilot (free) | **$0.00** | Trivial queries, greetings, simple lookups |
| **Standard** | MiMo 2.5 | OpenCode Go | $0.14 | Moderate tasks, file operations, straightforward coding |
| **Complex** | deepseek-v4-pro | OpenCode Go | $1.74 | Architecture design, debugging, multi-file changes |
| **Reasoning** | deepseek-v4-pro | OpenCode Go | $1.74 | Deep analysis, multi-step planning, complex reasoning chains |
| **Fallback** | GLM 5.1 | OpenCode Go | $1.40 | When primary tier is unavailable or rate-limited |

---

## Additional Model Assignments

| Role | Model | Why |
|---|---|---|
| **Delegation (subagents)** | deepseek-v4-flash | Fast and cheap — no reasoning waste on simple subagent tasks |
| **Vision** | MiMo-V2.5 | Via OpenCode Go auxiliary provider |
| **Loop Breaker** | gpt-5.3-codex | Dedicated safety model via OpenRouter |
| **Max Turns** | 90 per session | Prevents infinite loops while allowing complex multi-step work |

---

## The Philosophy

> **Free tier for simple. Paid model only when reasoning matters.**

This isn't just cost-cutting — it's intentional architecture:

- **~60-70% of messages are simple** — greetings, clarifications, single-tool calls. Burning reasoning tokens on these is wasteful.
- **Complex tasks need reasoning, not just bigger models** — deepseek-v4-pro is selected for its reasoning capability, not its parameter count.
- **Fallback ensures resilience** — if the primary provider has an outage, GLM 5.1 takes over automatically.
- **Delegation uses cheap models** — subagents do focused work; flash models handle it at 12x lower cost than reasoning models.

---

## Cost Comparison

If all 27 cron jobs + daily conversations ran on a single premium model at $15/M tokens, monthly costs would be substantial. With tiered routing:

| Task Type | % of Volume | Tier | Effective Cost |
|---|---|---|---|
| Simple queries | ~40% | Free | $0.00 |
| Standard tasks | ~35% | $0.14/M | Low |
| Complex/reasoning | ~20% | $1.74/M | Moderate |
| Fallback | ~5% | $1.40/M | Low |

The weighted average cost is dramatically lower than any single-model approach.

---

## Configuration

The router was configured directly by Shahriar — decisive, hands-on, no waiting for the agent:

```yaml
# Key settings in the Manifest Router config
tiers:
  simple:
    model: gpt-5.4-mini
    provider: copilot
  standard:
    model: mimo-2.5
    provider: opencode-go
  complex:
    model: deepseek-v4-pro
    provider: opencode-go
  reasoning:
    model: deepseek-v4-pro
    provider: opencode-go
  fallback:
    model: glm-5.1
    provider: opencode-go
```

---

## Lessons Learned

1. **Reasoning models need higher `max_tokens`.** deepseek-v4-flash consumes tokens for `reasoning_content` before producing visible output. At 500 tokens, responses were empty (`finish_reason: length`). Minimum 1024 tokens required.
2. **Provider keys are per-profile.** The market-channel profile needs its own `OPENCODE_GO_API_KEY` — it does not inherit from root `.env`.
3. **Fallback chains prevent silent failures.** When the primary provider returns 401 or 429, the router falls back automatically. Without this, cron jobs would silently produce nothing.
