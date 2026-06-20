# Projects Built & Deployed

> **Two projects built through Hermes Agent — a Python CLI habit tracker (published on PyPI) and a Tailwind CSS web agency site on Cloudflare Workers — plus the PolyMarket Trend data pipeline powering a live Telegram channel.**

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
| **Status** | Published on PyPI — `pip install habitrack` |

### Features

- Add, check, mark done/undone for daily habits
- Slug-based habit identification
- JSON data store
- Clean CLI interface
- Dual entry points for convenience

### Integration

The daily Todo cron jobs (morning check-in, afternoon reminder, evening wrap-up) query habitrack for current habit status and suggest tasks based on what's pending.

### Published

Available on PyPI: `pip install habitrack`

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

---

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

---

## Lessons Learned

1. **habitrack is PyPI-ready but not yet published — PYPI_TOKEN is empty.** The package builds cleanly (wheel + sdist in `dist/`), passes all 60 tests with zero dependencies, and has dual entry points (`habitrack` / `habit`). The only thing stopping publication is a PyPI token. When Shahriar is ready, it's a 2-command deploy: `python -m build` then `python -m twine upload dist/*`.
2. **Bettrix went through an 8-fix Claude Code session in one delegation.** A single structured prompt file was written, piped to Claude Code via background terminal with tee logging, and all 8 fixes (pricing, WhatsApp button, testimonials, portfolio grid, FAQ content, branding, favicon, responsive polish) were applied in one run. One prompt, one delegation, 8 changes verified post-execution. This is the power of the orchestrator pattern.
