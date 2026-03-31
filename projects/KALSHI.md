# KALSHI.md — Prediction Market Rules for Kowalski

> **Purpose:** Trading ruleset that Kowalski follows when operating in prediction market context. Not a standalone app — these are rules Kowalski applies when analyzing and trading event contracts on Kalshi.

---

## System Overview

- **Agent:** Kowalski (OpenClaw) operating under Kalshi rules
- **Interface:** BenAdmin console (chat + position log)
- **Platform:** Kalshi
- **Market Types:** CPI, GDP, Fed rate decisions, political/elections
- **Execution:** Fully automated — Kowalski enters and exits on its own

---

## API Configuration

> ⚠️ **No secrets in this file.** All API keys, tokens, and passwords live in `.env` on the Kowalski-v2 droplet.

### Kalshi (Trading Platform)
- **Provider:** Kalshi
- **API docs:** `https://trading-api.readme.io`
- **Base URL:** `https://trading-api.kalshi.com/trade-api/v2`
- **Auth method:** RSA key pair — `.env` → `KALSHI_API_KEY` (UUID) + `KALSHI_PRIVATE_KEY` (RSA PEM)
- **KALSHI_API_KEY:** `85a6d28e-1bc1-4f1b-a862-b2c0e36d69c4` ← store in .env, not here
- **KALSHI_PRIVATE_KEY:** RSA private key from Replit secrets — store in .env on droplet
- **Rate limits:** Check Kalshi docs
- **Websocket:** `[DECIDE: real-time pricing via WS?]`

### Data Sources (Research / Signals)
- **CPI forecasting:** `[DECIDE: e.g., Cleveland Fed Nowcast, BLS data, custom model?]`
  - URL: ___
  - Auth: `[DECIDE: public or keyed?]`
- **GDP forecasting:** `[DECIDE: e.g., Atlanta Fed GDPNow, custom model?]`
  - URL: ___
  - Auth: `[DECIDE: public or keyed?]`
- **Fed rate probabilities:** `[DECIDE: e.g., CME FedWatch, custom?]`
  - URL: ___
  - Auth: `[DECIDE: public or keyed?]`
- **Political data:** `[DECIDE: e.g., polling aggregators, other prediction markets?]`
  - URL: ___
  - Auth: `[DECIDE: public or keyed?]`

---

## 1. Market Selection

### Allowed Market Categories
- ✅ CPI / inflation data
- ✅ GDP data
- ✅ Fed rate decisions
- ✅ Political / elections
- ❌ Weather events
- ❌ Tech/corporate events
- ❌ All other categories

### Market Filters
- Minimum liquidity threshold: `[DECIDE: e.g., minimum open interest or daily volume]`
- Minimum time until event resolution: Must be within the 3–7 day entry window (see Section 5)

---

## 2. Entry Rules

### Pricing Rules
- **YES contract price range:** 10¢ – 85¢ only
  - Never buy YES above 85¢ (not enough upside)
  - Never buy YES below 10¢ (too speculative)
- **Edge threshold:** `[DECIDE: e.g., only enter if estimated edge is > X% vs. market price]`

### Signal Sources
- `[DECIDE: What triggers a Kalshi trade? Check all that apply]`
  - [ ] Data model / quantitative signal
  - [ ] Consensus vs. actual divergence play
  - [ ] Arbitrage between Kalshi pricing and other sources
  - [ ] Calendar-based (auto-position within entry window)
  - [ ] Other: ___

### Research Requirements
- **Before CPI/GDP/jobs markets:** `[DECIDE: e.g., check Cleveland Fed Nowcast, consensus estimates, recent trends]`
- **Before Fed rate markets:** `[DECIDE: e.g., check CME FedWatch, recent Fed speeches, economic data]`
- **Before political markets:** `[DECIDE: e.g., check polling aggregates, prediction market consensus]`
- **Minimum confirming signals:** `[DECIDE: e.g., at least 2 confirming data points before entering]`

---

## 3. Position Sizing

- **Max position per market:** $100
- **Max total Kalshi exposure:** $500 across all open positions
- **Max positions open at once:** 5 (implied by $500 total / $100 per)
- **Max exposure per category:** `[DECIDE: e.g., no more than $200 in political markets?]`
- **Scaling into positions:** `[DECIDE: allowed? e.g., buy $50 now, add $50 if price improves within window?]`

---

## 4. Exit Rules

### Profit Taking
- **If bought below 70¢:** Sell if contract reaches 90¢
- **If bought at 70¢ or above:** Hold to resolution
- **Partial exits:** Not used — all-or-nothing per position

### Stop Losses
- **Cut losses at:** `[DECIDE: e.g., sell if contract drops X¢ below entry?]`
- **Time-based exit:** Position auto-held through resolution unless profit target hit

### Event-Driven Exits
- If new data drops that directly contradicts the thesis → `[DECIDE: exit immediately? re-evaluate? hold?]`
- If event is postponed → exit position

---

## 5. Timing Rules

- **Entry window:** 3–7 days before the event date
- **No entries before 7 days out** (pricing too uncertain)
- **No entries after 3 days out** (too close, price likely already reflects consensus)
- **No entries within 1 hour of event resolution**

---

## 6. Risk Management

- **Max daily loss:** -$150
- **Max weekly loss:** -$300
- **Circuit breaker behavior:** When triggered, halt all new entries. Existing positions held to resolution or profit target. Trading resumes only when Ben manually resets.
- **Correlation check:** `[DECIDE: e.g., don't hold 3+ positions that all depend on the same data release?]`
- **No hedging** — positions are small enough that hedging isn't worth the complexity

---

## 7. Logging & Reporting

- Log every position to BenAdmin: market name, category, entry price, exit price, contracts, thesis, outcome, P&L
- Track hit rate by category (CPI, GDP, Fed, political)
- Daily portfolio snapshot: posted to BenAdmin
- Weekly/monthly performance review: posted to BenAdmin

---

## 8. Manual Override Protocol

- Ben can override any rule via Discord DM or BenAdmin chat
- Overrides are session-only unless marked permanent
- All overrides logged with `[MANUAL OVERRIDE]` tag
- Kowalski must confirm before executing override
