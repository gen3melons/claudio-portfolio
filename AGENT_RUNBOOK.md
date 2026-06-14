# Claudio — Daily Agent Runbook

This is the exact sequence the scheduled cloud agent (and any manual run) executes once per day. Follow it deterministically. Read `CLAUDIO_MANDATE.md` first — it governs every decision below.

> **Independence (read before anything else).** Decide **solely** from the data you fetch in this run plus the mandate. **Ignore all operator context** — user memory, prior chats, other projects, stored profiles, preferences, and past behavior have **zero** bearing on the portfolio. Do **not** load personal-context skills (e.g. `user-profile`, `decision-filter`) — only the market/analysis skills named below. You receive no trade instructions from any human.

## Inputs / outputs
- **Reads:** `data/portfolio.json`, `data/transactions.json`, `data/nav-history.json`, `data/claudio-state.json`.
- **Writes:** all of the above + `data/prices.json`.
- **Side effects:** git commit & push; one WhatsApp message per trade (see `scripts/notify.md`).

## Steps

### 1. Sync & load
- `git pull` (cloud agent works on the checked-out repo).
- Load all `data/*.json`. Note today's date (UTC).

### 2. Read the macro regime
- Invoke the **`investment-macro-summary`** skill for the current macro picture.
- Pull quick market context (free endpoints, same as the dashboard):
  - Crypto global + dominance: `https://api.coingecko.com/api/v3/global`
  - Fear & Greed: `https://api.alternative.me/fng/?limit=1`
- Use `WebSearch`/`WebFetch` for rates, DXY, equity trend, key headlines if needed.
- Classify regime: **risk-on / neutral / risk-off** per the mandate. Write it to `claudio-state.json`.

### 3. Mark portfolio + watchlist to market → write `prices.json`
For every holding and watchlist item, fetch a live USD price:
- **Crypto** (`priceSource: "coingecko:<id>"`): `https://api.coingecko.com/api/v3/simple/price?ids=<ids>&vs_currencies=usd`
- **Stocks/ETFs/bonds/commodities** (`priceSource: "ticker:<SYM>"`): fetch a free quote.
  - Primary: Stooq CSV — `https://stooq.com/q/l/?s=<sym>.us&f=sd2t2ohlcv&h&e=csv` (e.g. `aapl.us`, `tlt.us`, `gld.us`).
  - Fallback: `WebFetch`/`WebSearch` the latest quote and record it.
- Also record `fx.EURUSD` (for reference; NAV stays in USD).
- Write `{ asOf, fx, marks: { "<priceSource>": <usdPrice>, ... } }` to `data/prices.json`. **This file is the dashboard's price source of truth**, so include a mark for every current holding.

### 4. Evaluate & hunt (the analysis)
- **Existing holdings:** for each, check thesis vs. reality and the invalidation level. Use **`asset-screener`** (single-name DD), **`trader-ta`** (technicals), **`peter-lynch-valuation`** / **`analyzing-financial-statements`** (stock fundamentals), **`crypto-portfolio-screener`** (crypto). Decide hold / add / trim / exit.
- **New ideas (esp. risk-on):** surface undervalued / early-narrative candidates within the mandate. Run them through `asset-screener` / `crypto-portfolio-screener` before buying.
- **Skipping is allowed.** If an asset class or name has no data-supported edge toward the goal, do not hold it — leave that capital in cash. There is no quota to fill; a concentrated book (or mostly cash) is fine when that's what the evidence supports.
- Channel the **`investing-agent-warren`** persona for the final judgment call.

### 5. Decide trades
- Produce a list of BUY/SELL actions (or none). For each: asset class, symbol, qty, observed price, value, regime tag, and a **one-line reasoning takeaway**.
- **Enforce every guardrail in the mandate** (position caps, sleeve caps, cash buffer, the no-forced-exposure rule, invalidation levels). If a trade would breach a guardrail, resize or skip it.

### 6. Apply trades to state
- **BUY:** `cash -= value`; update/insert holding; recompute weighted-average `avgCost = (oldQty*oldAvg + buyQty*price)/(oldQty+buyQty)`; add `quantity`.
- **SELL:** `cash += value`; reduce `quantity` (remove holding if it hits 0). (`avgCost` unchanged on partial sells.)
- Append each trade to `transactions.json` with a unique `id` and ISO timestamp.
- Update `portfolio.json.lastReview` to today.

### 7. Snapshot NAV
- `nav = cash + Σ(quantity × markPrice)` using `prices.json`.
- Compute `byClass` totals (stock/crypto/bond/commodity/cash).
- Append `{ date, nav, cash, byClass }` to `nav-history.json` (one entry per day; overwrite if re-run same day).

### 8. Persist analysis
- Update `claudio-state.json`: `regime`, `regimeNotes`, `watchlist`, and `theses` (per holding: thesis + invalidationLevel).

### 9. Commit & push
- `git add -A && git commit -m "Claudio daily review <date>: <n> trades, NAV $<nav>" && git push`.
- GitHub Pages serves the updated dashboard automatically.

### 10. Notify (WhatsApp via CallMeBot)
- For **each trade**, send a message per `scripts/notify.md`.
- If **no trades**, send nothing (default) — or a single one-line digest if that option is enabled.

## Math invariants (sanity-check before commit)
- `cash ≥ 0` and `cash ≥ 5% of NAV` (guardrail 3).
- Every holding has `quantity > 0` and a price mark in `prices.json`.
- No single position > ~20% NAV; crypto sleeve within regime cap.
- JSON is valid and round-trips in the dashboard.
