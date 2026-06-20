# Backup Strategy & Service Resilience

> **Daily GitHub autobackup at 02:00 UTC with 14-snapshot rolling retention, biweekly gateway restarts for memory hygiene, and three isolated gateway services with auto-restart — the system heals itself.**

---

## GitHub Autobackup

| Detail | Value |
|---|---|
| **Repo** | Shahriar-008/hermes-backup (private) |
| **Format** | `hermes backup --quick` (compact snapshot tree) |
| **Schedule** | Daily at 02:00 UTC |
| **Retention** | Last 14 snapshots (rolling) |
| **Auth** | x-access-token via `.env` |
| **Script** | `github-hermes-backup.sh` |

### How It Works

```bash
# Pseudocode of the backup process
1. Clone Shahriar-008/hermes-backup (or pull if exists)
2. Run hermes backup --quick → creates timestamped snapshot
3. git add + git commit + git push
4. Prune old snapshots (keep last 14)
5. Clean up temporary clone
```

### Evolution

| Version | Format | Size | Note |
|---|---|---|---|
| **v1 (initial)** | Full ZIP | ~74 MB | Too large for GitHub, slow to push |
| **v2 (current)** | Quick snapshot | Slimmer | Directory-tree format, faster |

The switch from full ZIP to quick-snapshot was made after a single run proved the ZIP approach too heavy.

---

## Gateway Resilience

### Three Gateway Services

| Service | Profile | Auto-Restart |
|---|---|---|
| `hermes-gateway` | default | Yes |
| `hermes-gateway-group-chat` | group-chat | Yes |
| `hermes-gateway-market-channel` | market-channel | Yes |

All run as `systemd --user` services with `Restart=on-failure`.

### Biweekly Restart

| Detail | Value |
|---|---|
| **Schedule** | Monday and Thursday at 04:00 UTC |
| **Type** | no_agent cron script |
| **Purpose** | Prevent memory leaks from accumulating |

Long-running Python processes can develop slow memory growth. The biweekly restart is a proactive hygiene measure — it restarts before memory becomes a problem.

```bash
# The restart script (simplified)
systemctl --user restart hermes-gateway
systemctl --user restart hermes-gateway-group-chat
systemctl --user restart hermes-gateway-market-channel
```

---

## Resilience Architecture

```
┌────────────────────────────────────────────┐
│            RESILIENCE LAYERS                │
│                                            │
│  ┌──────────────────────────────────┐     │
│  │  systemd --user auto-restart     │     │
│  │  Gateway comes back if it dies   │     │
│  └──────────────────────────────────┘     │
│                  ▲                         │
│  ┌──────────────────────────────────┐     │
│  │  Biweekly cron restart           │     │
│  │  Proactive memory hygiene        │     │
│  └──────────────────────────────────┘     │
│                  ▲                         │
│  ┌──────────────────────────────────┐     │
│  │  Daily GitHub autobackup          │     │
│  │  Config + skills + cron snapshots │     │
│  └──────────────────────────────────┘     │
│                  ▲                         │
│  ┌──────────────────────────────────┐     │
│  │  Error handler + cron_errors.log │     │
│  │  Silent failure → DM + log       │     │
│  └──────────────────────────────────┘     │
└────────────────────────────────────────────┘
```

---

## What Gets Backed Up

| Path | Content |
|---|---|
| `~/.hermes/config.yaml` | Main configuration |
| `~/.hermes/skills/` | 71 installed skills |
| `~/.hermes/scripts/` | Watchdog, backup, utility scripts |
| `~/.hermes/cron/jobs.json` | Cron job definitions |
| Profile directories | Per-profile config, skills, scripts, cron |
| SOUL.md files | Agent personalities |

> **NOT backed up:** `.env` files (contain API keys), session transcripts (privacy), logs (high volume, low value).

---

## Recovery Scenario

If the VPS is lost or the Hermes installation is corrupted:

1. **Clone** `Shahriar-008/hermes-backup`
2. **Restore** snapshot to `~/.hermes/`
3. **Recreate** `.env` from secure storage (API keys)
4. **Reinstall** Hermes Agent: `curl -fsSL https://.../install.sh | bash`
5. **Restart** gateway services: `systemctl --user restart hermes-gateway`
6. **Verify** cron jobs: `hermes cron list`

Estimated recovery time: **15-20 minutes** for a full restore.

---

## Lessons Learned

1. **Full ZIP backups are too heavy for GitHub.** 74 MB pushes are slow and hit GitHub's file size limits. Quick-snapshot format is the right approach.
2. **Biweekly restarts prevent slow memory death.** Python processes that run for weeks develop leaks. A scheduled restart is simpler than debugging memory profiles.
3. **systemd --user with linger is critical.** Without `loginctl enable-linger`, services die on SSH logout.
4. **Backup verification matters.** The autobackup cron confirms the push succeeded. A silent backup failure is worse than no backup (false sense of security).
5. **`.env` must stay out of backups.** Secrets in version control are a liability. The backup captures structure; secrets are restored separately.
6. **The quick-snapshot format was adopted after a single full-ZIP run proved too heavy.** At ~74 MB, the ZIP backup was slow to push, risked GitHub's file size limits, and wasted bandwidth. Switching to `hermes backup --quick` produced a compact directory-tree snapshot that is faster, smaller, and more practical for daily automated backups. The lesson: test your backup strategy with real data before automating it.
