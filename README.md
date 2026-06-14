# Claudio — Autonomous Simulated Investing Portfolio

A **simulated** (paper) portfolio managed daily by an AI persona, **Claudio**, a pro-investor blending Buffett (quality/value), Dalio (macro regime + diversification), and active narrative/rotation trading. Real live prices, simulated trades, real discipline.

- **Capital:** $100,000 USD → **Goal:** $500,000 in 5–10 years
- **Universe:** stocks/ETFs · crypto · bonds · commodities · cash
- **Cadence:** daily review (cloud, 24/7)
- **Alerts:** WhatsApp on every trade (CallMeBot)

> Not financial advice. Education / tracking only.

## How it works

```
┌────────────────────────┐         ┌──────────────────────────┐
│  Claudio (cloud agent)  │  daily  │   data/*.json (state)     │
│  AGENT_RUNBOOK.md       │ ──────► │  portfolio, transactions, │
│  CLAUDIO_MANDATE.md     │  writes │  nav-history, prices,     │
│  + investing skills     │         │  claudio-state            │
└──────────┬─────────────┘         └────────────┬─────────────┘
           │ git push                            │ reads
           │ WhatsApp per trade                  ▼
           ▼                          ┌──────────────────────────┐
     📱 your phone                    │  index.html (dashboard)   │
                                      │  served via GitHub Pages  │
                                      └──────────────────────────┘
```

- **The brain:** [AGENT_RUNBOOK.md](AGENT_RUNBOOK.md) defines the deterministic daily loop; [CLAUDIO_MANDATE.md](CLAUDIO_MANDATE.md) defines philosophy + hard risk guardrails. Claudio orchestrates the investing skills (`investment-macro-summary`, `asset-screener`, `crypto-portfolio-screener`, `peter-lynch-valuation`, `analyzing-financial-statements`, `trader-ta`, `investing-agent-warren`).
- **The state:** plain JSON in `data/` (the single source of truth). Claudio is the only writer.
- **The face:** [index.html](index.html) — single-file React + Chart.js. Reads `data/*.json`, marks holdings to market, shows **Overview** (NAV, goal progress, allocation pie, holdings table) and **Transactions** (full log with reasoning).
- **Alerts:** [scripts/notify.md](scripts/notify.md) — CallMeBot WhatsApp recipe.

## Run it locally
```bash
# from this folder — any static server works
npx serve -p 5173 .
# open http://localhost:5173
```

## Deploy (24/7)
1. Push this folder to a GitHub repo.
2. Settings → Pages → serve from root (`/`). Note your Pages URL.
3. Create a daily cloud routine with the `/schedule` skill, bound to the repo:
   > "Execute Claudio's daily cycle exactly per AGENT_RUNBOOK.md and CLAUDIO_MANDATE.md. Use CallMeBot creds PHONE=… APIKEY=… for a WhatsApp alert per trade."
4. Complete CallMeBot setup (see [scripts/notify.md](scripts/notify.md)).

## Data files
| File | Purpose |
|------|---------|
| `data/portfolio.json` | cash + holdings (qty, avgCost, assetClass, priceSource) |
| `data/transactions.json` | append-only trade log with reasoning takeaways |
| `data/nav-history.json` | daily NAV snapshots + allocation by class |
| `data/prices.json` | latest price marks (dashboard source of truth) |
| `data/claudio-state.json` | regime read, watchlist, per-holding theses |
