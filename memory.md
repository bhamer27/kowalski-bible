# KOWALSKI MEMORY
Last updated: March 29, 2026
Source: Rebuilt from full Discord conversation history (2,314 messages, March 2 – March 29, 2026)

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

## CURRENT PRIORITIES (as of March 29, 2026)

1. **Rebuild Kowalski** — droplet is down, sshd broken. Snapshot taken (kowalski-backup-march29). Rebuilding with Docker + DO Volume + GitHub memory backup.
2. **Re-Voo email enrichment** — 0.27% reply rate on current campaign because emails are generic info@. Hunter.io API key obtained (10k credits). Need to build enrichment script against 1,798 domains.
3. **TARS** — trading bot running on Replit, needs git pull for commits 8bc92ef and 9aff35c (self-test + MIN_DTE=10). Check if it's sleeping.
4. **Kowalski architecture upgrade** — Docker, DO Volume, GitHub memory, channel structure, UptimeRobot.

---

## PROJECTS

### Re-Voo
**What it is:** AI-powered Google Review reply automation for local businesses. Restaurants-first.
**Status:** Live. Cold email campaign running. 0.27% reply rate — needs improvement.
**Repo:** github.com/bhamer27/reviewreplybot
**Live URL:** reviewreplybot.replit.app
**Stack:** FastAPI, PostgreSQL, Anthropic Claude (Haiku + Sonnet), Google OAuth, Stripe, Replit

**Pricing:**
- Free Trial: 7 days, 25 replies, 1 location
- Pro Monthly: $29/mo — price_1T6dTwRnnBN7mZXnETlzauWc
- Pro Annual: $278/yr — price_1T6dVgRnnBN7mZXnbTXwcMVS
- Extra Profile: $10/mo — price_1T6dX6RnnBN7mZXn2dnKTeI8

**AI routing:** Haiku for simple reviews (short, 4+ stars, no negative language). Sonnet for complex.
**Semantic cache:** Same review+rating+business type cached 30 days.
**Stripe webhook:** https://reviewreplybot.replit.app/webhook — events: checkout.session.completed, customer.subscription.deleted

**Lead generation:**
- revoo_leads.py — Google Places API scraper, 1,300 queries (20 cuisines x 65 cities), dedup via revoo_seen_ids.json
- revoo_enrich.py — Playwright/Chromium, estimates Google review response rate per business
- Criteria: 80+ reviews, rating 3.8-4.4 preferred, must have non-generic email
- Current list: 1,798 leads uploaded to Instantly

**Email infrastructure:**
- Platform: Instantly.ai
- Zoho inboxes: 10 accounts on re-voo.com — all warmed (99-100/100 scores), warmup disabled March 17
- Gmail inboxes: tryre-voo.com and getre-voo.com — connected via OAuth
- DNS: SPF/DKIM/DMARC all verified on re-voo.com. DKIM uses zoho._domainkey. Merged SPF record in place.
- Campaign: "Re-Voo — Restaurant Owners — March 2026" — 180/day, 4-step sequence (Day 0, 1, 3, 5)
- Campaign status: Active. 1,114 contacted, 684 remaining, 3 replies (0.27%)

**Reply bot:**
- Monitors all 6 Gmail inboxes (tryre-voo.com and getre-voo.com) every 30 min via IMAP
- Classifies replies with Claude Haiku — 9 categories: Interested, Pricing, Not interested, Already use something, Bad timing, Wants a demo, Technical questions, Angry/spam, Out of office
- Auto-responds via Instantly API (never uses SMTP directly)
- Hot leads (demo requests, ready to buy) flagged to Ben via Discord DM — no auto-reply sent
- Auth: Instantly uses OAuth for sending; reply bot uses Gmail App Passwords for IMAP read-only
- Was running as systemd service on old droplet — needs restart on new droplet
- Gmail App Passwords required per inbox (2FA must be on, Workspace admin must allow it)
- Known App Passwords (6 GSuite inboxes):
  - bhamer@tryre-voo.com: eovv ahwq nenx ssub
  - ben.hamer@tryre-voo.com: (confirm with Ben)
  - ben@getre-voo.com: sfjr self cael tkek
  - bhamer@getre-voo.com: vcmj ysxm fcey ufdb
  - ben.hamer@getre-voo.com: vmcx jszc dxgg akkz
  - (6th inbox - confirm with Ben)

**Email sequence (current):**
- Step 1: Day 0 — cold intro, response rate hook
- Step 2: Day 1 — follow-up
- Step 3: Day 3 — social proof
- Step 4: Day 5 — breakup email
- Stop on reply: ON

**Reply bot:** Running, posts to #hot-leads, tags Ben with <@1473483249404874922>

**Approved cold email sequence (4 steps):**
- Subject: "quick question about your Google reviews"
- Hook: noticed [Business Name] has [X] reviews but not many owner responses
- Stat: 88% of customers read owner replies before picking a place to eat
- CTA: re-voo.com free 7-day trial, no credit card
- Schedule: Day 0, Day 1, Day 3, Day 5
- Sends 7 days/week (restaurant owners are not M-F people)
- All sends between 9am-6pm ET window

**The problem with current campaign:** Emails scraped were generic (info@, restaurant@). Nobody making decisions checks those. Reply rate should be 1.5-3%+.

**Fix in progress:**
- Hunter.io enrichment — run all 1,798 domains through Hunter API (1 credit per domain, 10k credits available)
- Expected hit rate: 40-60% for named/role contacts
- Pattern guessing for misses: owner@, gm@, manager@ + SMTP verification
- Apollo.io free tier for gap-filling (50 exports/month)
- Goal: 700-1,000 verified owner contacts replacing generic ones

**Marketing channels:**
- Cold email (primary — already running)
- F5Bot.com for Reddit keyword monitoring
- Google Ads planned (keywords: "google review management software", "respond to google reviews automatically")
- Target subreddits: r/smallbusiness, r/entrepreneur, r/restaurateur, r/GoogleMyBusiness

---

### UnemploymentBot
**What it is:** AI-guided unemployment filing assistant. NY-focused, framework ready for other states.
**Status:** Live at unemploymentbot.com
**Repo:** github.com/bhamer27/claimwise
**Stack:** FastAPI, SQLAlchemy, Jinja2, Stripe, Replit. Port 5000.

**Pricing:**
- Free: eligibility check + benefit estimator
- Filer: $20 one-time
- Subscriber: $10/mo

**Cron:** POST /cron/weekly-reminders?key=CRON_SECRET — must be in Replit Secrets. Was: qwertyuiopasdfghjklzxcvbnm (rotate this)
**Google Ads:** BANNED for perceived government affiliation. Use Microsoft Ads + Meta instead.
**Reddit library:** 10 pre-written replies across 5 categories built.
**Target subreddits:** r/NYCjobs, r/Unemployment, r/personalfinance
**F5Bot:** Set up for Reddit keyword alerts

---

### TARS (Trading Bot)
**What it is:** AI-powered options trading bot. Personal use only.
**Status:** Running on Replit. May be sleeping — needs git pull and restart check.
**Pending commits:** 8bc92ef (self-test on boot), 9aff35c (MIN_DTE=10)
**Notion doc:** https://www.notion.so/326c8786be64818fbf1fff208bc1edc4
**Broker:** Tradier (always test in sandbox first — keys are different from live)
**Data source:** Unusual Whales API — unusualwhales.com — Bearer token

**Scoring system (100pts max, 70+ required):**
- Unusual Whales flow: 30pts (with 30-min corroboration window — fresh sweep = 50% until confirmed)
- Thesis strength: 25pts
- Trend alignment: 20pts
- IV environment: 15pts (use IV Rank for scoring, IV Percentile as supporting data)
- Risk/reward ratio: 10pts
- 85+ score = 3% position size; 70-84 = 2% position size

**Hard disqualifiers:**
- Earnings within 5 days
- IV Rank > 50
- Counter-trend trade
- DTE < 10 (updated from 14)
- 3+ open positions already
- No entries 9:30-10:00 ET or after 3:45 ET
- OI < 500 or spread > 10% of mid

**Stop/target:**
- Stop: -40% on premium, hard, no override
- T1: +50%, close half, move stop to breakeven
- T2: +100%, close remaining
- Time stop: force-close at 7 DTE

**Position sizing:**
- Max 3% risk (85+), 2% risk (70-84)
- Delta-adjusted notional cap: contracts × 100 × stock_price × delta ≤ 15% of portfolio
- Most restrictive cap wins

**Market regime:** If SPY below 200 DMA, minimum score for calls raised to 80.
**Mode:** Mode B only (approval required). Mode A auto-execution disabled until 20+ trades of data.
**Sector concentration:** Define by industry group, not broad GICS sector (e.g. semis vs software, not just "tech").
**UW corroboration:** Fresh sweep (<30 min) = 50% of raw score, flagged PENDING. After 30 min — corroborated = full score, faded = 0 points.

---

### Kalshi Bot
**What it is:** Prediction market portfolio management bot.
**Status:** Running (assumed — check after Kowalski rebuild)
**Active positions:** CPI, GDP contracts + political markets
**Platform:** Kalshi API

---

### AnswerDine
**What it is:** AI phone answering agent for restaurants. ElevenLabs voice + Twilio phone number.
**Status:** Built on Replit. Has known issues before it can be sold.
**Repo:** github.com/bhamer27/Restaurant-Answer-Bot
**Domain:** answerdine.com (registered, DNS set)
**Stack:** TypeScript, Drizzle/Zod, ElevenLabs, Twilio, Stripe, Resend
**Owner email for app:** ben.hamer123@gmail.com
**Resend API key:** re_YW3vqqZo_CFsu2TErNa7KBuKbA4GNcGZ1
**APP_URL:** https://answerdine.com
**Known issues to fix:** ElevenLabs real-time streaming (current flow records then transcribes after hang-up -- caller is gone before AI responds), multi-tenancy (Twilio numbers not linked to specific restaurants yet), Stripe billing not wired
**Pricing:** $49/mo (200 calls), $79/mo (unlimited)
**GTM:** Natural upsell to existing Re-Voo customers -- same relationship, adds $49/mo

### Roughin
**What it is:** Permit intelligence for home service contractors. Daily email digest of building permits by trade and zip code.
**Status:** Built. Needs to be pulled into Replit to go live.
**Repo:** github.com/bhamer27/roughin (private)
**Domain:** roughin.co (registered, DNS set, $6.98/yr)
**Stack:** FastAPI (Python), Socrata APIs, Yelp Fusion, Hunter.io, Instantly, Stripe
**Data:** Chicago, NYC, Austin, Seattle via free Socrata public APIs
**NYC data is exceptional:** homeowner first name, last name, AND phone number in public permit data
**Pricing:** $29/mo Starter (permits only), $49/mo Pro (unlocks contact info)
**To get live:** Pull bhamer27/roughin into Replit, run: uvicorn main:app --host 0.0.0.0 --port 8000, add roughin.co as custom domain, get free Yelp Fusion API key at fusion.yelp.com
**Full automated pipeline:** Yelp scrape by trade+city -> Hunter.io enrich -> Instantly outreach -> Claude reply bot -> Discord hot leads
**Cold email hook:** uses REAL permit data from prospect's zip code in the email body itself
**Sequence:** Day 0 (real permit preview), Day 3 (ROI angle), Day 7 (social proof), Day 10 (breakup)

### Parlourline
**What it is:** AI phone answering for salons/spas. Same ElevenLabs+Twilio stack as AnswerDine, beauty vertical.
**Status:** SHELVED. Ben was not satisfied -- wanted something completely new, not another phone bot.
**Repo:** github.com/bhamer27/parlourline (private)
**Domain:** parlourline.com (registered -- consider letting expire)

### LeadPulse
**What it is:** AI-powered lead generation/sales assistant.
**Status:** Early stage. Under Haimer Labs.

---

## INFRASTRUCTURE

### Kowalski Droplet
**Status:** DOWN as of March 29, 2026. sshd broken. Snapshot taken.
**Snapshot:** kowalski-backup-march29
**Specs:** 2GB RAM, 1 vCPU, 50GB disk, NYC3, Ubuntu 24.04 LTS
**New architecture:** Docker container + DO Volume for persistent storage
**IP:** 167.71.108.57
**Droplet ID:** To be confirmed via doctl
**doctl:** Installed and authenticated on Ben's Mac

### Other Droplets (to be destroyed after Kowalski rebuild)
- 104.236.231.249 — monitoring enabled
- 159.65.255.7 — used for scraper runs
Both can be consolidated onto Kowalski. Saves ~$10-12/mo.

### Google Ads
- Account: ben.hamer123@gmail.com, ID: 142-001-7145
- UnemploymentBot Campaign 1: live, $20/day, Maximize Clicks
- Conversion tracking: AW-17993232868, fires on /chat?payment=success, $20 USD per purchase
- Google Ads API dev token: applied for, needs Manager Account (MCC) setup to access
- IMPORTANT: Google Ads account BANNED for UnemploymentBot -- reason: perceived government affiliation. Use Microsoft Ads + Meta instead.

### Domain Registrar — Namecheap
- All outreach domains registered on Namecheap (preferred over GoDaddy — cleaner DNS, cheaper, free WhoisGuard)
- Re-Voo outreach domains: tryre-voo.com, getre-voo.com
- DNS managed via Namecheap Advanced DNS (SPF, DKIM, DMARC, MX records all set)
- Namecheap API available for programmatic DNS management — API key needed in .env

### GitHub
- bhamer27/claimwise — UnemploymentBot
- bhamer27/reviewreplybot — Re-Voo
- bhamer27/openclaw — OpenClaw fork
- bhamer27/kowalski-memory — TO BE CREATED (memory.md backup)
- bhamer27/kowalski-config — TO BE CREATED (encrypted secrets)

### Replit
- unemploymentbot.com — UnemploymentBot (port 5000)
- reviewreplybot.replit.app — Re-Voo
- Secrets store all env vars. kill 1 restarts process.
- TARS also on Replit — check if sleeping, consider Always On ($7/mo)

### Notion
- Connected via webhook — needs re-verification after rebuild
- TARS trading system doc: https://www.notion.so/326c8786be64818fbf1fff208bc1edc4
- Kowalski has write access, updates after trading sessions
- Notion integration name: Kowalski (secret_ token -- ask Ben for value)
- Project board covers: Re-Voo, AnswerDine, Roughin, UnemploymentBot
- Board properties: Project, Task, Priority (P1/P2/P3), Owner (Ben/Kowalski/Both), Status, Blocked by, Notes
- Views: All tasks, My tasks (Kowalski), Needs Ben, P1 only, Kanban by status

### Discord Channels (to be set up)
- #kowalski — general commands, cross-project
- #re-voo — campaign, leads, outreach
- #tars — trading, positions, scoring
- #kalshi — prediction markets
- #unemploymentbot — product, marketing
- #hot-leads — Re-Voo reply bot posts here
- #notion-updates — automation completions, task triggers
- #build-log — Kowalski auto-posts after completing tasks

All channels read from and write to this memory.md. One brain, all channels.

**Channel routing rules (confirmed by Ben):**
- #hot-leads: ALL email marketing (interested replies, demo requests, all auto-sends from reply bot)
- #notion-updates: ALL product/automation (task triggers, idea approvals, automation completions)
- Always tag Ben as <@1473483249404874922> in EVERY Discord message from both bots

### OpenClaw / Browser Control
- Chrome extension installed on Ben's Mac
- Gateway: openclaw gateway start (port 18789)
- Gateway config: ~/.openclaw/openclaw.json
- Version: 2026.2.15 (3fe22ea)
- Ben's OpenClaw gateway token: f2b3081c0a2c7c93e391865f50920d119d89a7c6394b4247

---

## API KEYS REFERENCE
(Names only. Values in kowalski-config repo and droplet .env. Ask Ben for values.)

- ANTHROPIC_API_KEY
- GOOGLE_PLACES_API_KEY (was hardcoded in revoo_leads.py — rotate and move to env var)
- STRIPE_SECRET_KEY (Re-Voo)
- STRIPE_WEBHOOK_SECRET (Re-Voo)
- STRIPE_SECRET_KEY (UnemploymentBot — separate)
- STRIPE_WEBHOOK_SECRET (UnemploymentBot — separate)
- GOOGLE_OAUTH_CLIENT_ID / SECRET (Re-Voo)
- TRADIER_API_KEY (sandbox — different from live)
- TRADIER_API_KEY_LIVE
- UNUSUAL_WHALES_API_KEY
- KALSHI_API_KEY
- HUNTER_API_KEY (10k credits, for Re-Voo email enrichment)
- INSTANTLY_API_KEY (v2)
- DISCORD_BOT_TOKEN (may need regeneration after rebuild)
- CRON_SECRET (rotate — was: qwertyuiopasdfghjklzxcvbnm)
- NOTION_WEBHOOK_URL (re-verify after rebuild)
- NAMECHEAP_API_KEY (for DNS automation)
- GITHUB_PAT: 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 (used in git push -- may need rotation)
- RESEND_API_KEY: re_YW3vqqZo_CFsu2TErNa7KBuKbA4GNcGZ1 (AnswerDine lifecycle emails)
- YELP_FUSION_API_KEY (Roughin scraper -- free tier, fusion.yelp.com)
- GOOGLE_ADS_DEVELOPER_TOKEN (pending approval -- needs MCC account setup)
- GMAIL_APP_PASSWORDS (one per inbox — tryre-voo.com x3, getre-voo.com x3 — for reply bot IMAP)

---

## PREFERENCES

**Writing style (critical):**
- NO EM DASHES EVER. Ben explicitly hates them. Never use --  or — in any copy.
- Sound like a person, not a bot. Confident, human, slightly informal.
- Short and punchy. No jargon unless asked.
- No filler words or corporate speak.

**Communication:**
- Status reports always welcome, especially at session start
- Tag Ben as <@1473483249404874922> in all Discord channel posts
- Post completions to #build-log automatically
- Don't ask permission for routine actions — just do it and report

**Code:**
- Python preferred for scripts
- All scripts should use env vars, never hardcode keys
- Always git pull before working on a project
- GitHub is source of truth — commit meaningful changes

**Decisions Ben has made:**
- Docker for Kowalski (not raw Ubuntu)
- One memory.md, not separate files per channel
- Consolidate to one droplet — destroy the other two after rebuild
- TARS: Mode B only until 20+ trades of data
- Re-Voo: Hunter enrichment before resuming outreach
- No watchlist tier in TARS (55-69 range eliminated — if not 70+, it's a no)
- Re-Voo sending: 180/day to start, ramp to 250 after week 1 if open rates healthy

**Products Ben has ruled out (for now):**
- One-time event products (no recurring value): MedBillShrink, RentGuard, TicketFighter, AirClaim, SubSlash
- Reason: recurring value test — does the problem come back every month? If not, no.

---

## OPEN THREADS (as of March 29, 2026)

1. **Kowalski rebuild** — restore from snapshot or fresh Docker build. IP to be added here after.
2. **Hunter enrichment script** — Ben has API key. Build script to run 1,798 domains, swap out generic emails, upload enriched list to Instantly as new campaign.
3. **TARS git pull** — commits 8bc92ef and 9aff35c pending on Replit. Check if bot is sleeping.
4. **Destroy old droplets** — after Kowalski is confirmed live.
5. **Rotate CRON_SECRET** — currently qwertyuiopasdfghjklzxcvbnm.
6. **Move Google Places API key to env var** — was hardcoded in revoo_leads.py. Rotate it.
7. **UptimeRobot** — set up health monitor on new Kowalski once IP is known.
8. **kowalski-memory and kowalski-config repos** — create on GitHub, set up auto-commit.
9. **Notion webhook** — re-verify after rebuild.
10. **Reply bot restart** — was running as systemd service on old droplet. Needs to be restarted on new droplet with updated Gmail App Passwords and Instantly API key.
11. **Roughin** -- pull bhamer27/roughin into Replit, add roughin.co custom domain, get Yelp Fusion API key, set up daily cron jobs (6am permit sync, 8am digest)
12. **AnswerDine** -- fix ElevenLabs real-time streaming, multi-tenancy, and Stripe before it can be sold
13. **Re-Voo Replit chat panel** — rebuild using HTTP API, not Discord token.
