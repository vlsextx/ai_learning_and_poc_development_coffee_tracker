# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Family Coffee Tracker — a PoC three-tier web app that tracks coffee stock, logs brew sessions, predicts depletion, sends email reminders, and forecasts monthly spending. Single shared family account (no per-member auth for PoC). See README.md for full specs.

## Commands

```bash
npm install          # install dependencies
node index.js        # run entry point (not yet implemented)
```

No build, lint, or test scripts are configured yet (`npm test` exits with an error by default).

## Planned architecture

### Tech stack
- **Frontend**: React + Vite (SPA, four screens: Dashboard, Stock, Log Brew, Stats)
- **Backend**: Node.js (Express) or Python (FastAPI) — REST API
- **Database**: PostgreSQL (production) / SQLite (local dev)
- **Email**: SMTP or SendGrid
- **Scheduler**: node-cron or APScheduler (daily cron for email reminders)
- **AI integration**: `@anthropic-ai/sdk` is already installed — Claude API usage is planned

### Database schema (five tables)
- `coffee_products` — catalogue (brand, name, roast, grind)
- `purchases` — each shopping trip (weight, price, date, shop)
- `bag_sessions` — lifecycle of each bag (opened, finished, status)
- `brew_logs` — optional per-cup log (cups made, brewed_by)
- `reminders` — history of sent email reminders

Relationship chain: `coffee_products → purchases → bag_sessions → brew_logs / reminders`

### Key backend logic
- **Consumption estimate**: `g_per_day = avg(cups_per_day) × avg(g_per_cup)`, derived from last 3–5 completed bag sessions. Falls back to bag-level averages when brew logs are absent.
- **Spending forecast**: `base_cost = kg_per_month × avg_price_per_kg`, adjusted by month-over-month price trend, with ±15% confidence band.
- **Email reminder cron**: fires daily; sends if `remaining_g / avg_g_per_day ≤ threshold` (default 5 days) and no reminder was sent recently.

## Environment

Copy `.env.example` to `.env` for local config (`.env` is gitignored). Expected variables will include DB connection, SMTP/SendGrid credentials, and Anthropic API key.
