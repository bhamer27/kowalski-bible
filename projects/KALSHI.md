# KALSHI.md — Prediction Market Rules for Kowalski

> **Purpose:** Trading ruleset that Kowalski follows when operating in prediction market context. Not a standalone app — these are rules Kowalski applies when analyzing and trading event contracts on Kalshi.

---

## System Overview

- **Agent:** Kowalski (OpenClaw) operating under Kalshi rules
- **Interface:** BenAdmin console (chat + position log)
- **Platform:** Kalshi
- **Market Types:** CPI, GDP, Fed rate decisions, political/elections
- **Execution:** Automated — Kowalski enters and exits on its own
- **Starting bankroll:** $500

---

## API Configuration

> ⚠️ **No secrets in this file.** All API keys, tokens, and passwords live in `.env` on the Kowalski-v2 droplet.

### Kalshi (Trading Platform)
- **Provider:** Kalshi
- **API docs:** `https://trading-api.readme.io`
- **Base URL:** `https://trading-api.kalshi.com/trade-api/v2`
- **Auth method:** RSA key pair — `.env` → `KALSHI_API_KEY` (UUID) + `KALSHI_PRIVATE_KEY` (RSA PEM)
- **KALSHI_API_KEY:** `85a6d28e-1bc1-4f1b-a862-b2c0e36d69c4` ← store in .env only
- **KALSHI_PRIVATE_KEY:** RSA private key stored in `/root/.openclaw/kalshi_private.pem`
- **Websocket:** Not used — polling only

### Data Sources (Research / Signals)
- **CPI forecasting:** Cleveland Fed Nowcast (public, no key required) — `https://www.clevelandfed.org/indicators-and-data/inflation-nowcasting`
- **GDP forecasting:** Atlanta Fed GDPNow (public) — `https://www.atlantafed.org/cqer/research/gdpnow`
- **Fed rate probabilities:** CME FedWatch (public) — `https://www.cmegroup.com/markets/interest-rates/cme-fedwatch-tool.html`
- **Political data:** Polymarket consensus + polling aggregates (public)
- All signal sources are public/free — no additional API keys required

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
- Minimum open interest: 1,000 contracts
- Must be within the 3–7 day entry window (see Section 5)

---

## 2. Entry Rules

### Pricing Rules
- **YES contract price range:** 10¢ – 85¢ only
  - Never buy YES above 85¢ (risk/reward too poor)
  - Never buy YES below 10¢ (too speculative — lottery ticket territory)

### Edge Threshold
- Only enter if estimated probability edge is >5% vs. current market price
- Example: model says 65% chance YES, contract priced at 55¢ = 10¢ edge = enter

### Signal Sources — what triggers a Kalshi trade
- Consensus vs. actual divergence play (primary) — model disagrees with market price
- Data model signal — Cleveland Fed, GDPNow, CME FedWatch pointing away from consensus
- Calendar-based — within entry window with clear directional signal

### Research Requirements
- **Before CPI/GDP markets:** Check Cleveland Fed Nowcast and GDPNow vs. consensus. Minimum 2 confirming data points.
- **Before Fed rate markets:** Check CME FedWatch + recent Fed speeches + economic data trend. Minimum 2 confirming signals.
- **Before political markets:** Check polling aggregates + Polymarket cross-reference. Minimum 2 confirming signals.

---

## 3. Position Sizing

- **Max position per market:** $100
- **Max total Kalshi exposure:** $500 across all open positions (entire bankroll)
- **Max positions open at once:** 5 (implied by $500 total / $100 per)
- **Max exposure per category:** $200 in political markets (more volatile/subjective)
- **Scaling into positions:** Not allowed — one entry, full size

---

## 4. Exit Rules

### Profit Taking
- **If bought below 70¢:** Sell if contract reaches 90¢ — lock in gains, don't wait for resolution
- **If bought at 70¢ or above:** Hold to resolution
- **Exits are all-or-nothing** — no partial exits

### Stop Losses
- **Cut losses:** Sell if contract drops 50% from entry price
  - Example: bought at 40¢ → sell if drops to 20¢
- **Time-based exit:** Hold through resolution unless profit target or stop hit

### Event-Driven Exits
- If new data directly contradicts the thesis → exit immediately, don't average down
- If event is postponed → exit position

---

## 5. Timing Rules

- **Entry window:** 3–7 days before the event date
- **No entries before 7 days out** (pricing too uncertain, too much time to reverse)
- **No entries after 3 days out** (priced in, spread to resolution vs. edge is too small)
- **No entries within 1 hour of event resolution**

---

## 6. Risk Management

- **Max daily loss:** –$150 (30% of bankroll)
- **Max weekly loss:** –$300 (entire bankroll, triggers full halt)
- **Circuit breaker behavior:** When triggered, halt all new entries. Existing positions held to resolution or stop. Trading resumes only when Ben manually resets.
- **Correlation check:** No more than 2 positions that depend on the same data release
- **No hedging** — positions are small enough that hedging isn't worth the complexity

---

## 7. Logging & Reporting

- Log every position to BenAdmin: market name, category, entry price, exit price, contracts, thesis, outcome, P&L
- Track hit rate by category (CPI, GDP, Fed, political)
- Daily portfolio snapshot: posted to BenAdmin
- Weekly performance review: posted to BenAdmin

---

## 8. Manual Override Protocol

- Ben can override any rule via Discord DM or BenAdmin chat
- Overrides are session-only unless marked permanent
- All overrides logged with `[MANUAL OVERRIDE]` tag
- Kowalski must confirm before executing override

<!-- Updated: 2026-03-31 -->
