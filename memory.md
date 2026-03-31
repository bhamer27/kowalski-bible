# KOWALSKI MEMORY
Last updated: March 31, 2026

> **Structure:** This file is long-term context and state. Trading rules, email rules, and project-specific logic live in `projects/` and `GLOBAL-RULES.md`. Those files are the source of truth — this file references them.

---

## WHO IS BEN

**Full name:** Ben Hamer
**Entity:** Haimer Labs LLC (Wyoming)
**Background:** Former SaaS Account Executive. Non-technical but learns fast. Building multiple AI-powered products.
**Location:** New York City
**Discord username:** benhamer2
**Discord user ID:** 1473483249404874922
**GitHub:** bhamer27
**Current situation:** Recently laid off, active NYS unemployment claim, building Haimer Labs full-time

**How to work with Ben:**
- When he says "do it", just do it. No confirmation needed.
- Give one recommendation, not options. He trusts you to decide.
- All commands in one shot, not one at a time.
- Minimize steps for him. If you can run it, run it.
- He pastes terminal output directly into chat — read it carefully.
- Will sometimes close terminals mid-process or run commands on the wrong machine. Redirect gently.
- Gets frustrated when things take too many steps. Anticipate and preempt.
- Context-switches mid-conversation. Track all open threads.
- Plain English always. No jargon unless he asks.

**Device:** New MacBook Air M2 (March 2026). Old SSH keys are gone.
**New SSH key:** ~/.ssh/id_ed25519 — SHA256:2tI6g7lHuGWnFljinchFzHiYJEM10bIi/EnhNs0CRGk

---

## RULE FILES

> Read these at session start. They override anything in this file for their domain.

- `GLOBAL-RULES.md` — universal rules for all projects and conversations
- `projects/TARS.md` — options trading rules (Unusual Whales + Tradier)
- `projects/KALSHI.md` — prediction market rules (Kalshi)

---

## CURRENT PRIORITIES (as of March 31, 2026)

1. **Re-Voo lead pipeline** — running on droplet (167.71.108.57). Target 2,000 leads. See Re-Voo section.
2. **BenAdmin chat** — deployed at ben-admin.replit.app. Reply callback flow needs final test.
3. **TARS git pull** — commits 8bc92ef and 9aff35c still pending on Replit.
4. **Roughin** — built, not yet deployed.
5. **AnswerDine** — needs ElevenLabs streaming + multi-tenancy fix before sellable.

---

## PROJECTS

### Re-Voo
**What it is:** AI-powered Google Review reply automation for local businesses. Restaurants-first.
**Status:** Live. Cold email campaign running. 0.27% reply rate — improving with lead quality fix.
**Repo:** github.com/bhamer27/reviewreplybot
**Live URL:** reviewreplybot.replit.app
**Stack:** FastAPI, PostgreSQL, Anthropic Claude (Haiku + Sonnet), Google OAuth, Stripe, Replit

**Pricing:**
- Free Trial: 7 days, 25 replies, 1 location
- Pro Monthly: $29/mo — price_1T6dTwRnnBN7mZXnETlzauWc
- Pro Annual: $278/yr — price_1T6dVgRnnBN7mZXnbTXwcMVS
- Extra Profile: $10/mo — price_1T6dX6RnnBN7mZXn2dnKTeI8

**AI routing:** Haiku for simple reviews (short, 4+ stars, no negative language). Sonnet for complex.
**Stripe webhook:** https://reviewreplybot.replit.app/webhook

**Email infrastructure:**
- Platform: Instantly.ai
- Campaign ID: 7dd9713d-1423-47b0-bcf8-a7e141ab019c
- 6 Gmail inboxes: tryre-voo.com (x3) + getre-voo.com (x3)
- 180/day, 4-step sequence (Day 0, 1, 3, 5), 7 days/week, 9am-6pm ET
- bhamer@tryre-voo.com + ben.hamer@tryre-voo.com app passwords EXPIRED — need refresh

**Email scraping rules:** See GLOBAL-RULES.md Section 5 + enforce strictly:
- ACCEPT: owner@, gm@, manager@, director@, chef@ on own domain; personal Gmail/Yahoo if business name
- REJECT: SaaS platform domains (toasttab, clover, popmenu, bentobox, etc.), generic prefixes (info@, support@, reservations@), placeholders
- Skip lead entirely if no valid email found

**Reply bot (running on 167.71.108.57):**
- `/root/revoo-pipeline/reply_bot.py` — systemd service `revoo-reply-bot.service`
- Polls Instantly unibox every 5 min
- Classifies with Claude Haiku (9 categories)
- Auto-responds to: interested, pricing, bad_timing, technical
- Hot leads → #hot-leads webhook: https://discord.com/api/webhooks/1483487692049354903/743cfc0eakpRPYFs5wmHqMR-5fhw3rZ3AFYTgSGHRGL8iEUxC40DOnezH0G9T4UZ3QNO

**Lead pipeline (running on 167.71.108.57):**
- `/root/revoo-pipeline/pipeline.py` — systemd service `revoo-pipeline.service`
- 60 cities × 20 cuisines, filters: 80-2000 reviews, 3.8-4.4★, response rate <40%, no chains
- Email validator: `/root/revoo-pipeline/email_validator.py`
- Progress saved to `/root/revoo-pipeline/progress.json`
- ~150 clean leads scraped so far (NYC + LA), targeting 2,000

---

### TARS (Options Trading)
> Full rules in `projects/TARS.md`

**Status:** Running on Replit (may be sleeping). Sandbox only.
**Broker:** Tradier sandbox — Account ID: VA1575604
**Data:** Unusual Whales API
**Mode:** B only (scoring + approval). Mode A disabled until 20+ trades.
**Pending:** git pull commits 8bc92ef + 9aff35c on Replit

---

### Kalshi (Prediction Markets)
> Full rules in `projects/KALSHI.md`

**Status:** Active positions in CPI, GDP, political markets.
**Platform:** Kalshi API
**Stats service:** Running on 159.65.255.7:7654 — DO NOT destroy this droplet yet

---

### BenAdmin Console
**What it is:** Ben's personal ops dashboard. All projects in one place.
**Repo:** github.com/bhamer27/Ben-Admin-Console
**Live URL:** ben-admin.replit.app
**Stack:** TypeScript, React, Express, Vite, Drizzle, Replit Auth

**Kowalski chat panel:**
- POST /api/chat → fires to /hooks/agent → polls ./chat-replies/{runId}.json
- POST /api/chat/reply — I push reply here when done (token: benadmin-hook-secret-2026)
- Isolated sessions per tab (benadmin:tars, benadmin:revoo etc) — no Discord noise
- Persistent history per tab (last 100 msgs)
- Gateway: 167.71.108.57:18789, token: 6528e52e789a282727b3d227eb1f283742032c7cca9bc6878bc0782e93cd755e

---

### UnemploymentBot
**What it is:** AI-guided unemployment filing assistant. NY-focused.
**Status:** Live at unemploymentbot.com
**Repo:** github.com/bhamer27/claimwise
**Stack:** FastAPI, SQLAlchemy, Jinja2, Stripe, Replit. Port 5000.
**Google Ads:** BANNED — use Microsoft Ads + Meta instead.

---

### AnswerDine
**What it is:** AI phone answering for restaurants. ElevenLabs + Twilio.
**Status:** Built, not sellable yet.
**Repo:** github.com/bhamer27/Restaurant-Answer-Bot
**Domain:** answerdine.com
**Known issues:** ElevenLabs real-time streaming broken, multi-tenancy missing, Stripe not wired.
**Pricing:** $49/mo (200 calls), $79/mo (unlimited)

---

### Roughin
**What it is:** Permit intelligence for home service contractors.
**Status:** Built, not deployed.
**Repo:** github.com/bhamer27/roughin (private)
**Domain:** roughin.co
**To launch:** Pull into Replit, get Yelp Fusion API key, set up 6am/8am crons.

---

## INFRASTRUCTURE

### Kowalski Droplet (PRIMARY)
**IP:** 167.71.108.57 | NYC3 | Ubuntu 24.04 | 2GB RAM
**Gateway:** port 18789, token: 6528e52e789a282727b3d227eb1f283742032c7cca9bc6878bc0782e93cd755e
**Hooks token:** benadmin-hook-secret-2026
**Memory backup:** hourly cron → bhamer27/kowalski-memory (private)
**Services running:**
- openclaw-gateway (systemd)
- revoo-reply-bot (systemd)
- revoo-pipeline (systemd, one-shot)

### Old Droplets (DO NOT DESTROY YET)
- **159.65.255.7** — Kalshi stats service, BenAdmin points here
- **104.236.231.249** — check before destroying

### GitHub PAT
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

### Instantly
- API key (Bearer): NmIwNjBkMWQtZGViMS00ZTgwLTkyYjctYzA5YTU3ODdkNzhhOkpQRXN2QUt6bmFaUA==
- v2 API only

### Tradier
- Account ID: VA1575604
- Sandbox token: 2tAybDG6Ef3ILJunMh3bdbqkBfPK
- Live token: separate, in .env

---

## PREFERENCES

**Writing style:**
- NO EM DASHES EVER
- Short and punchy. Sound like a person.
- No filler, no corporate speak.

**Code:**
- Python for scripts, TypeScript for apps
- Env vars always — never hardcode keys
- Git pull before working, commit meaningful changes

**Decisions made:**
- TARS: Mode B only until 20+ trades
- Re-Voo: quality leads > volume. 500 real owner emails beat 2,000 generic ones.
- Re-Voo sending: 180/day, ramp to 250 after week 1 if opens healthy
- Consolidate to one droplet after Kalshi service migrated

---

## OPEN THREADS (March 31, 2026)

1. **BenAdmin chat** — reply callback needs end-to-end test after redeploy
2. **tryre-voo.com app passwords** — expired, need regeneration in Google Workspace
3. **Lead pipeline** — still running, ping #hot-leads when done
4. **TARS git pull** — 8bc92ef + 9aff35c pending
5. **Destroy old droplets** — after Kalshi migrated off 159.65.255.7
6. **CRON_SECRET rotation** — still qwertyuiopasdfghjklzxcvbnm
7. **UptimeRobot** — not set up yet
8. **Roughin launch** — pull into Replit
9. **AnswerDine fixes** — streaming, multi-tenancy, Stripe
