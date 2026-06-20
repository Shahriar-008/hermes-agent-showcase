# Projects Built & Deployed

> **Three projects built through Hermes Agent — a Python CLI habit tracker (open-source ready), a Tailwind CSS web agency site on Cloudflare Workers, and the Tech It Easy brand (AI Agents, Chatbots, Micro SaaS).**

---

## 1. habitrack — CLI Habit Tracker

| Detail | Value |
|---|---|
| **Path** | `/root/work/habit-tracker/` |
| **Language** | Python 3 |
| **Package** | pip-installable (`pip install habitrack`) |
| **Entry Points** | `habitrack` / `habit` (dual) |
| **Module** | `habitrack` |
| **Tests** | 60 pytest tests (all passing) |
| **Dependencies** | **Zero external deps** |
| **Build** | `dist/` ready (wheel + sdist) |
| **Status** | Built, tested, open-source ready |
| **Blocking** | `PYPI_TOKEN` empty — not yet published to PyPI |

### Features

- Add, check, mark done/undone for daily habits
- Slug-based habit identification
- JSON data store
- Clean CLI interface
- Dual entry points for convenience

### Integration

The daily Todo cron jobs (morning check-in, afternoon reminder, evening wrap-up) query habitrack for current habit status and suggest tasks based on what's pending.

### Next Step

Publish to PyPI once `PYPI_TOKEN` is configured. The package is fully ready — tests pass, `dist/` is built, README is written.

---

## 2. Bettrix — Web Agency Website

| Detail | Value |
|---|---|
| **Path** | `/root/work/bettrix/` |
| **Type** | Pure static site (HTML, CSS, JS) |
| **Design** | Dark-mode redesign with Stitch aesthetic |
| **CSS** | Tailwind CSS 3 compiled via PostCSS |
| **Pages** | 6 (index, about, contact, portfolio, pricing, services) |
| **Deploy** | Cloudflare Workers (`npx wrangler deploy`) |
| **Live URL** | [bettrix.shahriar-hj2022.workers.dev](https://bettrix.shahriar-hj2022.workers.dev/) |
| **GitHub** | [github.com/Shahriar-008/bettrix](https://github.com/Shahriar-008/bettrix) |

### Features

| Feature | Description |
|---|---|
| **Tiered Pricing** | $199 / $399 / $699 plans with real feature differentiation |
| **Floating WhatsApp Button** | Direct contact channel |
| **Testimonials** | Improved social proof section |
| **Portfolio Grid** | Card-based project showcase |
| **FAQ** | Specific timelines and deliverables |
| **Branding** | Custom favicon + nav logo styling |

### Development Method

Built and iterated entirely through Hermes Agent + Claude Code delegation:

1. Initial build via prompt-file delegation to Claude Code
2. Dark-mode redesign session — comprehensive prompt → Claude Code background run with tee
3. 8-fix polish session — all changes verified post-execution
4. Deployed via `npx wrangler deploy` to Cloudflare Workers

---

## 3. Tech It Easy — Brand & Business

| Detail | Value |
|---|---|
| **Focus** | AI Agents, Chatbots, Micro SaaS |
| **Founder** | Shahriar Hussain |
| **Location** | Narayanganj, Bangladesh |
| **X/Twitter** | [@ShahJoy23](https://x.com/ShahJoy23) |

Tech It Easy is the umbrella brand under which all projects operate. The philosophy:

> *Build small, ship fast, automate everything.*

The Hermes Agent setup itself is the flagship demonstration — it proves the capability to build, deploy, and automate at production scale from a single VPS.

---

## Project Comparison

| | habitrack | Bettrix | PolyMarket Trend |
|---|---|---|---|
| **Type** | CLI tool | Static website | Data pipeline |
| **Language** | Python | HTML/CSS/JS | Python + Bash |
| **Deploy Target** | PyPI | Cloudflare Workers | Telegram channel |
| **Status** | Ready, pending token | Live | Live |
| **Built With** | Hermes + Claude Code | Hermes + Claude Code | Hermes + cron |
| **Open Source** | Planned | GitHub | Scripts available in backup |

---

## The Pattern

All three projects share the same development pattern:

1. **Idea** → Discussed with Hermes in DM
2. **Plan** → Hermes breaks down into sub-tasks
3. **Build** → Claude Code handles implementation via prompt-file delegation
4. **Verify** → Hermes checks results, runs tests, confirms deployment
5. **Automate** → Cron jobs handle ongoing maintenance/delivery

This is the orchestrator pattern in action — Hermes never becomes the bottleneck because work is delegated.
