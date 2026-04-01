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

---

## 10. Agent Architecture

> This section defines the multi-layer agent pipeline that replaces the single Python scoring loop. All existing rules (circuit breakers, PDT, position sizing, blackout periods) remain in full effect and are enforced at the Layer 1 filter stage before any LLM is invoked.

### Layer 1 — Dumb Filter (No LLM)

Pure Python pre-screen that runs against every incoming Unusual Whales signal before any agent is invoked. Runs in microseconds, costs nothing.

**Hard rules:**
- Minimum premium: $500k
- Exclude 0DTE unless earnings is the explicit thesis
- IV rank within acceptable range (per Section 1 scoring)
- Expiry window: 7–45 DTE only
- Underlying liquidity minimums: sufficient volume + bid/ask spread < 10% of mid
- No entry within 2 days of earnings unless earnings is the thesis
- All existing hard disqualifiers from Section 1 apply here

**Goal:** Reduce thousands of daily signals to dozens of candidates. LLM stages never see noise.

---

### Layer 2 — Haiku Screener

One Claude Haiku call per candidate that survived Layer 1.

**Haiku evaluates:**
- Flow pattern recognition (sweep vs block, repeat flow, institutional sizing)
- Red flag detection (unusually low premium-per-contract, suspicious timing, earnings adjacency)
- Overall conviction score: 1–10

**Advancement threshold:** Score 7+ advances to Layer 3. Score <7 is logged as SKIP with reasoning.

**Cost controls:**
- Haiku only — never Sonnet at this stage
- Each evaluation is max 3–5 lines of structured output, no raw feed data
- Macro regime signal cached hourly, reused across all ticker evaluations in that window

**Goal:** Dozens of candidates → 3–8 high-conviction setups per day.

---

### Layer 3 — Full 6-Stage Pipeline (Sonnet)

Only runs on candidates that cleared both Layer 1 and Layer 2. Sonnet is used here exclusively.

---

#### Stage 1: Parallel Signal Ingestion (5 agents, parallel)

Five agents run simultaneously, each owning exactly one data source:

| Agent | Source | Output |
|---|---|---|
| Flow Agent | Unusual Whales flow data | Signal strength, direction bias, sweep/block classification |
| IV Agent | IV rank + IV percentile | Premium environment, vol regime alignment |
| Macro Agent | SPY trend + VIX level | Regime confirmation, trend alignment |
| Catalyst Agent | Earnings calendar + event proximity | Binary risk, catalyst timeline |
| Dark Pool Agent | Dark pool prints + OI changes | Institutional positioning, confirmation/divergence |

Each agent returns structured JSON:
```json
{
  "signal_strength": 1-10,
  "direction_bias": "bullish|bearish|neutral",
  "confidence": 1-10,
  "rationale": "one line"
}
```

---

#### Stage 2: Adversarial Debate (2 agents, parallel)

Two agents run in parallel with no visibility into each other's output:

- **Bull Agent:** Builds the strongest possible case FOR the trade using Stage 1 outputs
- **Bear Agent:** Builds the strongest possible case AGAINST — signal conflicts, regime mismatch, low liquidity, overextended moves

Orchestrator receives both outputs. Neither agent's output alone determines the outcome — the orchestrator weighs both.

---

#### Stage 3: Risk Screener (sequential, runs before scenario modeling)

Hard constraint checks that cannot be overridden by any debate output:

- PDT status (rolling 5-day window)
- DTE on the specific contract
- Bid/ask spread viability at current market
- Portfolio-level Greeks exposure (delta concentration, theta burn rate)
- Existing correlated positions (same sector, same underlying, same catalyst)

**If any hard constraint is tripped: pipeline stops here. No exceptions.**

---

#### Stage 4: Scenario Modeler (1 agent)

One agent builds three outcomes using all prior stage data:

- **Bull case:** Price target, probability estimate, expected return
- **Base case:** Most likely path given current regime and signal strength
- **Bear case:** Max loss scenario, probability of hitting hard stop

Outputs a probability-weighted expected value (EV). **Negative weighted EV kills the trade regardless of Stage 2 outcome.**

---

#### Stage 5: Orchestrator (1 agent, final decision)

Ingests all Stage 1–4 outputs. Produces:

- **Go / No-go** decision
- **Contract selection:** Specific strike + expiry recommendation
- **Position size:** As % of portfolio (within Section 2 limits)
- **Entry trigger:** Market vs. limit order, specific price level if limit
- **Rationale:** One paragraph logged to memory.md and BenAdmin trade log

---

#### Stage 6: Post-Trade Monitor (async, runs continuously on open positions)

Lightweight async loop watching all open positions:

- Monitors IV crush risk as earnings approach
- Flags if underlying crosses stop threshold before hard stop triggers
- Recommends early exit if original thesis is invalidated by new flow data (e.g., large opposing sweep)
- Logs full outcome on close (entry, exit, P&L, thesis validation/invalidation)
- Feeds result back into Stage 1 signal weighting for future calibration

---

### Cost Summary

| Layer | Model | Frequency | Purpose |
|---|---|---|---|
| Layer 1 | None (Python) | Every signal | Noise elimination |
| Layer 2 | Haiku | Per L1 survivor | Quick conviction screen |
| Stage 1 | Haiku | Per L2 survivor | Parallel signal ingestion |
| Stages 2–5 | Sonnet | Per L1+L2 survivor | Full pipeline |
| Stage 6 | Haiku | Per open position | Async monitoring |

<!-- Updated: 2026-04-01 -->
