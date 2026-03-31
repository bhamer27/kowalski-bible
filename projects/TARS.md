# TARS.md — Options Trading Rules for Kowalski

> **Purpose:** Trading ruleset that Kowalski follows when operating in options trading context. TARS is not a standalone app — it is a set of rules Kowalski applies when analyzing Unusual Whales flow data and executing trades on Tradier.

---

## System Overview

- **Agent:** Kowalski (OpenClaw) operating under TARS rules
- **Interface:** BenAdmin console (chat + trade log + UW flow list)
- **Data Source:** Unusual Whales (UW) options flow API
- **Broker:** Tradier — **SANDBOX ONLY** until Ben promotes to live
- **Mode:** B (scoring-based signal generation)
- **Core Logic:** Score each UW flow alert → filter by thresholds → execute on Tradier sandbox → log to BenAdmin
- **Account Type:** Under $25K (PDT restrictions apply)
- **Starting capital (live):** ~$2,000

### What Kowalski Does Under TARS Rules
1. Ingests UW flow data and posts it to a running list in BenAdmin
2. Scores each flow alert using the scoring system below
3. If score meets threshold + all filters pass → executes trade on Tradier sandbox
4. Logs all trades, signals, and P&L to BenAdmin
5. Manages open positions (stops, profit targets, forced exits) per these rules
6. Reports daily summary to BenAdmin

---

## API Configuration

> ⚠️ **No secrets in this file.** All API keys, tokens, and passwords live in `.env` on the Kowalski-v2 droplet.

### Unusual Whales (Flow Data)
- **Provider:** Unusual Whales
- **Base URL:** `https://api.unusualwhales.com`
- **Data used:** Options flow alerts (real-time)
- **Auth method:** Bearer token in `.env` → `UW_API_KEY`
- **Rate limits:** Check UW docs — respect per-minute limits

### Tradier (Brokerage / Execution)
- **Provider:** Tradier
- **Environment:** SANDBOX (do not switch to live without explicit instruction from Ben)
- **Base URL (sandbox):** `https://sandbox.tradier.com/v1`
- **Base URL (live — NOT ACTIVE):** `https://api.tradier.com/v1`
- **Account type:** Margin, under $25K (PDT applies)
- **Auth method:** Bearer token in `.env` → `TRADIER_ACCESS_TOKEN`
- **Account ID:** `.env` → `TRADIER_ACCOUNT_ID` (VA1575604)
- **⚠️ Sandbox only** — live keys are separate, never use live without Ben's explicit go

### Market Data (Regime Detection)
- **VIX + SPY data source:** Tradier market data API (same token)
- **Refresh interval:** Every 5 minutes during market hours

---

## 1. Entry Rules

### Scoring System
- **Scale:** 0–100
- **Minimum score to trade (Low Vol regime):** 70+
- **Minimum score in Elevated Vol regime:** 80+

Scoring factors and weights:
- Flow premium size: up to 30pts ($1M+ = 30, $500K+ = 22, $250K+ = 15, $100K+ = 8, <$100K = 3)
- Sweep vs block: sweep = 15pts, block/split = 5pts
- Volume/OI ratio: 3x+ = 20pts, 1.5x+ = 12pts, 0.5x+ = 6pts
- DTE sweet spot (14-30): 15pts; 7-45: 8pts
- Premium per contract ($0.50-$5.00): 10pts; >$0.20: 5pts
- Elevated vol regime: multiply total by 0.9 (–10%)

### Allowed Trade Types
- ✅ Long calls
- ✅ Long puts
- ❌ All other strategies (spreads, naked, etc.)

Long-only = max loss is always exactly the premium paid. No margin risk, no assignment surprises.

### Ticker Filters
- **Allowed tickers:** S&P 500 components only
- **Minimum option OI:** 500 contracts

### Time Filters
- **DTE range:** 7–45 days (max 30 DTE in Elevated Vol regime)
- **No entries 9:30–10:00 AM ET** (market open noise)
- **No entries 3:30–4:00 PM ET** (close noise)

---

## 2. Position Sizing

- **Default per trade:** $200–$500 (Low Vol regime)
- **Elevated Vol regime:** $100–$250 (half size)
- **High Vol regime:** No new trades
- **Max positions open at once:** 3
- **Max positions per ticker:** 1
- **Max total exposure:** ~$1,500 (3 × $500)
- **Scaling into positions:** Not allowed — one entry per position

---

## 3. Exit Rules

### Profit Targets
- **Default profit target:** +50% gain (all regimes)
- **Elevated Vol:** +35% gain
- **Clean exit:** Hit target, close full position, free up slot

### Stop Losses
- **Hard stop:** –25% on premium, no exceptions, no override
- **Time-based exit:** Close any position reaching < 7 DTE

### Forced Exits
- **Close ALL positions by Friday EOD** — no weekend holding (gap risk)
- **Close before any blackout event** for that ticker

---

## 4. Regime Detection

### Inputs
- VIX level (real-time, from Tradier)
- SPY 20-day and 50-day moving averages (from Tradier)
- VIX term structure (front month vs second month)

### Regime 1: Low Vol / Trending
**Conditions:** VIX < 18 AND SPY above both 20MA and 50MA
- Full position sizing: $200–$500
- All trade types enabled (long calls, long puts)
- Score threshold: 70+
- Profit target: +50%
- DTE range: 7–45 days

### Regime 2: Elevated Vol / Choppy
**Conditions:** VIX 18–28, OR SPY between 20MA and 50MA
- Half position sizing: $100–$250
- Score threshold raised to: 80+
- Profit target tightened to: +35%
- Max DTE shortened to: 30 days

### Regime 3: High Vol / Crisis
**Conditions:** VIX > 28, OR SPY below both 20MA and 50MA
- **No new trades.** TARS sits on hands.
- Only allowed action: manage/close existing positions
- **Resume conditions:** VIX drops back below 25 AND SPY reclaims 20MA (both required)

---

## 5. PDT Protection

- **PDT protection:** ENFORCED (account under $25K)
- **Max day trades per rolling 5-day window:** 3
- **Behavior when at PDT limit:** Block all new entries that would require a same-day exit. Allow only swing trades (hold overnight minimum).
- **PDT override:** Not allowed. Hard rule.

---

## 6. Circuit Breakers

These halt ALL trading when triggered. Trading resumes only when Ben manually resets.

- **Max daily loss:** –$300
- **Max weekly loss:** –$750
- **Max consecutive losses:** 3 in a row → pause, alert Ben
- **Flash crash protection:** If VIX spikes >20% in 15 minutes → halt all entries

---

## 7. Blackout Periods

**No new entries during these events.** Evaluate existing positions for early exit.

- ✅ **FOMC announcement days** — no entries within 1 hour before or after
- ✅ **CPI release days** — no entries within 1 hour before or after
- ✅ **NFP / jobs report days** — no entries on release day
- ✅ **Earnings for the specific ticker** — no entries 1 trading day before through 1 trading day after
- ✅ **Market holidays / half days** — no entries on half days

---

## 8. Logging & Reporting

- **UW flow data:** Post every incoming flow alert to BenAdmin running list (regardless of score)
- **Signals:** Log every scored signal (even if not traded): timestamp, ticker, strike, expiry, score, reason for pass/fail, current regime
- **Trades:** Log every trade executed: entry time, entry price, exit time, exit price, P&L, hold duration, score at entry, regime at entry
- **Daily summary:** Post to BenAdmin at market close
- **Weekly performance report:** Post to BenAdmin every Friday after all positions closed

---

## 9. Manual Override Protocol

- Ben can override any rule in real-time via Discord DM or BenAdmin chat
- Overrides last for the **current session only** unless Ben says "make this permanent"
- All overrides are logged in BenAdmin trade log with `[MANUAL OVERRIDE]` tag
- Kowalski must confirm the override before executing: "Overriding [rule]. Proceeding with [action]. Confirm?"

<!-- Updated: 2026-03-31 -->
