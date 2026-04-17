# Coffee Tracker — PoC Concept

A three-tier family web application for tracking coffee consumption, predicting stock depletion, and forecasting monthly spending.

---

## Overview

Coffee Tracker helps a family understand their coffee habits. The app tracks purchased bags, logs brew sessions, estimates when stock will run out, sends timely email reminders to buy more, and provides statistics on consumption and cost over time.

---

## Goals

- Predict and notify when the family is about to run out of coffee
- Show how many cups per period were consumed
- Show how many kilograms of coffee beans were used
- Show total spend and cost per cup
- Forecast future shopping budget based on consumption trends

---

## Architecture

The application follows a classic three-tier architecture.

### Tier 1 — Frontend (React)

A single-page web application with four main screens:

- **Dashboard** — stock status with depletion progress bars, monthly summary metrics, and a shopping reminder with budget forecast
- **Stock** — detailed view of open and sealed bags, past bag history, and consumption rate per bag
- **Log brew** — forms for logging a new purchase and recording an optional brew session
- **Stats** — cups per week bar chart, per-cup cost, monthly bean consumption, and 6-month spending forecast

### Tier 2 — Backend (REST API)

A Node.js or Python server exposing REST endpoints. Responsibilities:

- Calculate average daily consumption from bag session history
- Estimate remaining stock and days until depletion
- Run the spending forecast algorithm (see below)
- Schedule and trigger email reminders via a daily cron job
- Serve aggregated statistics to the frontend

### Tier 3 — Database (PostgreSQL / SQLite)

Five tables store all persistent state:

| Table | Purpose |
|---|---|
| `coffee_products` | Catalogue of coffee varieties (brand, name, roast, grind) |
| `purchases` | Each shopping trip — weight, price, date, shop |
| `bag_sessions` | Lifecycle of each bag — opened, finished, status |
| `brew_logs` | Optional per-cup log — cups made, who brewed |
| `reminders` | History of scheduled and sent email reminders |

---

## Data model relationships

```
coffee_products ──< purchases ──< bag_sessions ──< brew_logs
                                        │
                                        └──< reminders
```

---

## Consumption logging

The bag is the primary tracking unit:

- Opening a bag starts a `bag_session`
- Finishing or discarding a bag closes it
- Per-cup `brew_logs` are optional — useful for per-member breakdowns but not required for forecasting

When brew logs are absent, the backend falls back to average consumption rate derived from completed bag sessions.

---

## Spending forecast algorithm

The forecast runs in three steps.

### Step 1 — Consumption estimate

```
g_per_day    = avg(cups_per_day) × avg(g_per_cup)
kg_per_month = g_per_day × 30.5 / 1000
```

`avg(cups_per_day)` is derived from the last 3–5 completed bag sessions: total bag weight divided by days open.

### Step 2 — Cost estimate

```
base_cost    = kg_per_month × avg_price_per_kg
trend        = month-over-month change in price_per_kg (from purchases history)
forecast     = base_cost × (1 + trend)
```

### Step 3 — Confidence range

```
low  = forecast × 0.85
high = forecast × 1.15
```

The ±15% band is fixed for the PoC. As more bag history accumulates, this can be replaced with statistically derived variance from actual consumption data. A linear regression on monthly costs provides a more principled trend line for future iterations.

---

## Email reminders

A daily cron job runs the following logic:

1. For each open bag session, compute estimated days remaining: `remaining_g / avg_g_per_day`
2. If days remaining ≤ configured threshold (default: 5 days), check whether a reminder has already been sent in the last N days
3. If not, generate and send an email via SMTP / SendGrid and log the reminder in the `reminders` table

The threshold is configurable per family account.

---

## User model

A single shared family account — no per-member authentication for the PoC. An optional `brewed_by` field on `brew_logs` allows per-member attribution without requiring separate logins.

---

## In scope (PoC)

- Bag purchase logging
- Stock tracking and depletion estimation
- Email reminders
- Cups per period and kg consumed statistics
- Monthly cost tracking and 6-month spending forecast

## Out of scope (future iterations)

- Multi-user authentication and per-member profiles
- Mobile push notifications
- Barcode / QR scanning of coffee bags
- External store or subscription integrations

---

## Tech stack (suggested)

| Layer | Technology |
|---|---|
| Frontend | React, Vite |
| Backend | Node.js (Express) or Python (FastAPI) |
| Database | PostgreSQL (production) / SQLite (local dev) |
| Email | SMTP or SendGrid |
| Scheduler | node-cron or APScheduler |
