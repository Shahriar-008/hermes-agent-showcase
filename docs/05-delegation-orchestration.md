# Delegation & Subagent Orchestration

> **Hermes operates as an orchestrator, not an executor — planning, delegating to subagents (Claude Code + leaf workers), and synthesizing results. This is the exact design pattern Hermes was built for.**

---

## The Orchestration Pattern

```
User describes goal
     │
     ▼
┌────────────────────┐
│   Hermes (main)    │  1. Plans the approach
│   Orchestrator     │  2. Delegates to subagents
└───────┬────────────┘  3. Synthesizes results
        │                4. Reports back
   ┌────┼────┬────────┐
   ▼    ▼    ▼        ▼
┌────┐┌────┐┌────┐┌────────┐
│ CC ││Leaf││Leaf││ Claude │
│    ││ 1  ││ 2  ││  Code  │
└────┘└────┘└────┘└────────┘
```

---

## Delegation Configuration

| Setting | Value | Rationale |
|---|---|---|
| **Model** | deepseek-v4-flash | Fast, cheap — no reasoning waste on simple subagent tasks |
| **Provider** | OpenCode Go | $0.14/M tokens |
| **Max Concurrent** | 3 children | Parallel independent workstreams |
| **Max Spawn Depth** | 1 | No nesting — every child is a leaf (no grandchildren) |
| **Orchestrator** | Disabled | Children can't spawn their own workers |

---

## What Gets Delegated vs What Stays

### Routinely Delegated
- Web searches and research synthesis
- File reading and summarization  
- Code implementation (Claude Code)
- Configuration changes
- Cron job creation
- Multi-file HTML/CSS edits
- Code review and cleanup

### Kept in Main Context
- Architecture decisions
- User communication
- Synthesis and verification
- Complex reasoning chains
- Cross-subagent coordination

> **The rule:** If a task requires 3+ tool calls with processing logic, delegate it. If it requires understanding the full conversation context, keep it.

---

## Claude Code Integration

Claude Code is the heavy-lifting subagent — used for multi-file code changes, complex implementations, and code review.

| Detail | Value |
|---|---|
| **Path** | `/root/.hermes/node/bin/claude` |
| **Primary Use** | Multi-file code changes, complex implementations, code review |
| **Pattern** | Prompt-file delegation |

### Prompt-File Delegation Pattern

1. Write a structured `.claude-task.md` with the full task specification
2. Pipe it to Claude Code via terminal
3. Verify all changes post-execution

**Modes:**
- **Print-mode one-shots** — Fast, single-file or small-scope changes
- **Background with tee+tail** — Long-running multi-file sessions. Logs written to file, tailed for verification.

### Real Example: Bettrix 8-Fix Session

Wrote a comprehensive prompt file → dispatched Claude Code via background terminal with tee → verified all 8 fixes post-execution. One prompt, one delegation, all changes done.

---

## Background Delegation

For long-running tasks that shouldn't block the conversation:

```python
terminal(
    command="claude < .claude-task.md",
    background=True,
    notify_on_complete=True
)
```

The result re-enters the conversation when completed. The user keeps working while the subagent runs.

---

## Delegation Discipline

### The Watchdog

A Python watchdog (`delegation_watchdog.py`) runs every 5 minutes, checking the LCM database for tool call patterns:

| Metric | Threshold | Action |
|---|---|---|
| Direct:Delegate ratio | >5:1 | Telegram alert + stop_signal.json |

### The Self-Audit

Every 3 hours, an LLM-driven cron job scans DM sessions, counts tool calls, compares delegation ratios, and delivers a structured report with CRITICAL/WARNING/INFO tiers.

### Why This Matters

Without discipline, the orchestrator falls back to doing work directly — reading files one by one, searching manually, executing commands step by step. A single `delegate_task()` call replaces 5-25 direct tool calls. The watchdog enforces the pattern.

---

## Lessons Learned

1. **One delegate_task can replace 25+ direct calls.** A session analysis found the agent doing 25+ direct tool calls for what should have been one delegation. The watchdog caught it.
2. **Background delegation prevents context bloat.** Long subagent output pollutes the main context. Background mode + synthesis keeps things clean.
3. **Prompt-file delegation is more reliable than inline prompts.** Structured `.claude-task.md` files with explicit acceptance criteria produce better results than ad-hoc inline instructions.
4. **Subagent summaries are self-reports.** Always verify subagent claims — fetch the URL, stat the file, read back the content.
