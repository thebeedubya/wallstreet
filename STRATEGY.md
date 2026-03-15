# Wall Street Agent — Strategy & Architecture

## Overview

Automated arbitrage detection and execution using secretary-tier agents running 24/7 on local hardware (Kona/Runtz). Thinker-tier (Claude API) evaluates trade quality before execution. Google Fiber provides latency advantage over retail competitors.

---

## The Thesis: AI-Leveraged Retail

This is NOT an institutional strategy. We are not competing with Citadel head-on. We are a retail trader with AI leverage, operating in markets institutions won't touch, collecting crumbs that are life-changing at our scale.

### Why Retail + AI Wins Here

| Institutional Constraint | Our Advantage |
|-------------------------|---------------|
| Compliance blocks prediction markets | No compliance department |
| Legal blocks sports betting arb | Own risk tolerance |
| AML policy blocks crypto-forex bridge | Can operate in gray areas |
| $10M minimum position or don't bother | $500 position is meaningful |
| 200-person committee to approve strategy | Decide in 5 minutes |
| Can't use LLMs for trade decisions (infosec) | Claude/GPT is our co-pilot |
| Quarterly reporting, investor drawdown limits | Answer to nobody |

### The AI Leveling Effect

Two years ago, earnings call NLP required a $2M/year team. Today: frontier models do it better than most custom models, transcripts are free (SEC EDGAR), local inference on 128GB VRAM is unlimited. The gap between institutional and retail analytical capability closes with every model generation. We don't need to match them. We need to be 80% as good at 0.01% of the cost, in markets they won't enter.

### The Crumbs Strategy

None of these modules are "viable institutional strategies." All of them are viable for one person with AI and no compliance overhead. DFV didn't beat Citadel with better infrastructure — he beat them with better conviction and asymmetric positioning. Our version: AI provides the analysis, we pick spots, we size for crumbs at their scale but real money at ours, and we do it 24/7 with agents that never sleep.

---

## Consortium Pressure Test (March 15, 2026)

Five-model roundtable (Claude, GPT-5.4, Gemini 2.5 Pro, Grok 4, DeepSeek v3.1) unanimously pressure-tested this strategy. Full transcript: `data/consortium/consortium-2026-03-15T03-11-49.md`

### What They Got Right — Incorporated

**1. Unit Economics Per Trade (GPT)**
Before projecting returns, every module must track:
- Signal source and latency requirement
- Expected edge per trade
- Fees, slippage, and fill probability per trade
- Max deployable capital before edge degrades
- Failure modes and kill switch conditions

**2. "Detected Edge vs Captured Edge" (GPT)**
The only metric that matters. Track the full waterfall:
```
Gross theoretical spread
→ Tradable spread at order submission
→ Realized spread after fills
→ Realized spread after fees
→ Realized spread after operational loss
```
If captured edge is negative, the module dies regardless of how many opportunities the scanner "finds."

**3. Risk Infrastructure Before Scaling**
Before deploying serious capital, build:
- Global kill switch (one command shuts everything down)
- Per-module daily loss limits
- Per-venue exposure limits
- Stale data detection (exclude prices older than 60s)
- Duplicate order prevention
- Audit logs for every trade
- PnL attribution by module, venue, and cost bucket
- Tax lot tracking across asset classes

**4. Module 6 Is Highest Ceiling**
All five models agreed: AI Earnings Intelligence has the most durable, scalable edge. It's the only module where AI leverage is maximum and the edge is analytical, not speed-based. Priority: build this deepest.

**5. Start Narrow, Prove, Expand**
Don't build all 6 simultaneously. Sequence them. Prove each module's captured edge before adding the next. But don't kill any — they're all viable at retail scale.

### What the Research Says (The Consortium Didn't Look)

The consortium opined from training data. We did the actual research. The cutting edge contradicts their core assumptions.

**Prediction Market Arb — Proven, Not Theoretical:**
- $40M in arb profits extracted from Polymarket alone, April 2024 - April 2025 (IMDEA Networks Institute, arXiv:2508.03474)
- One automated bot: 8,894 trades, ~$150K profit, zero human intervention. 1.5-3% per trade on completeness gaps (CoinDesk, Feb 2026)
- Wall Street quants actively moving INTO prediction markets for arb (FinanceMagnates)
- Prediction market arb specialists being hired at $200K salaries
- Volumes: $100M/month (early 2024) → $8B/month (Dec 2025). More volume = more mispricing.
- Open source arb bot exists: github.com/realfishsam/prediction-market-arbitrage-bot

**Earnings Call NLP — NOT Solved, Newly Cracked Open:**
- S&P Global published "From Lexicon to LLM" (Sep 2025) — the entire field is being rewritten with frontier LLMs. BERT-era approaches are obsolete.
- LSEG (London Stock Exchange Group) building LLM-based earnings analysis for institutional alpha
- Key finding: overall transcript sentiment is USELESS. Segment-level sentiment (by business unit) predicts price moves. This changes Module 6 architecture entirely.
- ECC Analyzer (arXiv:2404.18470): hierarchical extraction from earnings calls using RAG for volatility prediction
- MarketSenseAI 2.0 (arXiv:2502.00415): RAG + LLM agents processing SEC filings + earnings calls
- 192,000 earnings call transcripts analyzed with LLM embeddings to quantify CEO transparency
- "Firms with high sentiment (top 10%) during earnings calls have significant next-month outperformance" — published, peer-reviewed signal

**AI Trading Bots — Retail IS Making Money:**
- Top AI trading platforms showing 12-25% annualized returns
- Some platforms achieving 40%+ annualized with profit factors over 4.0 (Tickeron, Trade Ideas)
- Global AI trading market: $24.53 billion in 2025
- 2026 evolution: bots are now AI Agents scanning X/news, adjusting in real-time

### Applied Research Insights

**1. Segment-Level Sentiment (S&P Global):**
Don't analyze the whole transcript as one blob. Break earnings calls into business segments. Score each segment independently. The alpha is in segment-level divergence from expectations, not overall tone.

**2. Hierarchical RAG Extraction (ECC Analyzer):**
Use retrieval-augmented generation to extract paragraph-level signals + fine-grained focus sentences. Don't feed the whole transcript to Claude — extract structured features first, then reason over them.

**3. Post-Call Drift Window (1-5 days):**
The market prices headline numbers (EPS, revenue) in seconds. But nuance — guidance softening, segment weakness, supply chain signals — takes 1-5 days to fully price. That's the window. Don't race speed. Race interpretation quality.

**4. Per-CEO Language Baselines (LSEG / 192K transcripts):**
LLM embeddings across 10 years of a specific CEO's calls create a "normal" baseline. Deviation from that baseline IS the signal. "Encouraged" from Satya ≠ "encouraged" from Pichai. The model must know each executive's personal language fingerprint.

**5. Look-Ahead Bias Guard (arXiv):**
LLMs trained on data including future information can leak it into predictions. When backtesting Module 6, use models with knowledge cutoffs BEFORE the earnings date being tested. Pin model versions. This is a known failure mode the papers warn about.

**6. Completeness Arb at Scale (IMDEA):**
$40M proven. 1.5-3% per trade. 8,894 trades automated. This is not theoretical — it's documented, peer-reviewed, and happening right now.

### What They Got Wrong — Rejected

**"You can't compete because HFT exists"** — We're not competing with HFT. We're operating in markets they won't touch (prediction markets, sports betting, exotic crosses at $10K scale). Their compliance departments are our moat.

**"Return projections are delusional"** — At institutional scale, yes. At retail scale collecting crumbs across 6 uncorrelated strategies 24/7 with zero labor cost, the math works differently. $500-2000 per arb trade, 5-10x per week, across multiple modules = real money.

**"Pick one thing, spend 12 months studying"** — Institutional thinking. Retail moves fast, takes small positions, learns by doing with real money, and lets AI close the knowledge gap in hours, not years.

**"Hyperspace AGI is a scam"** — Maybe. But contributing idle VRAM to a network costs nothing and returns either useful signals or nothing. Zero downside, potential upside. We don't bet the strategy on it — we run it as a free option on idle compute.

**"Save your 750K"** — The entire consortium evaluated this as an institutional strategy competing with Renaissance. That's the wrong frame. We're a wallstreetbets persona with AI behind it. The crumbs they leave are our feast.

---

## Module 1: Prediction Market Arbitrage (First Build)

### Cross-Platform Arb

Buy YES on Platform A + Buy NO on Platform B. If combined cost < $1.00, guaranteed profit.

| Platform A | Platform B | Why It Works |
|-----------|-----------|-------------|
| **Polymarket** | **Kalshi** | Same events, different user bases. Polymarket is crypto-native (volatile pricing), Kalshi is CFTC-regulated (tighter spreads). Structural divergence. |
| **Polymarket** | **Robinhood** | Robinhood just entered prediction markets. New platform = inefficient pricing. Retail flow = dumb money creating spreads. |
| **Polymarket** | **PredictIt** | PredictIt has 850-contract cap + 10% profit fee = artificial price distortion. Slow to update. |
| **Polymarket** | **Betfair** | Deepest sports/politics exchange globally (UK). Cross-border price differences on elections, geopolitics. |

**APIs:**
- Polymarket: On-chain (Polygon), public APIs, websocket feeds
- Kalshi: REST API with websocket feeds
- Robinhood: REST API (prediction markets section)
- Betfair: Exchange API (requires UK account or VPN consideration)

### Intra-Platform Contract Completeness Arb

Mutually exclusive outcomes on a single platform **must** sum to $1.00. When they don't, free money.

**Example — "Who wins the 2028 GOP primary?"**

| Candidate | Price |
|-----------|-------|
| DeSantis | $0.35 |
| Vance | $0.25 |
| Haley | $0.15 |
| Ramaswamy | $0.08 |
| "Other" | $0.10 |
| **Total** | **$0.93** |

Buy all contracts for $0.93. One **must** pay $1.00. Profit: $0.07 risk-free.

**Why it happens:**
- New candidates added, prices don't rebalance instantly
- Panic selling oversells the field
- Low-liquidity tail contracts get stale
- More outcomes = more surface area for mispricing (15-candidate market drifts constantly)

**Reverse:** If sum > $1.00 (say $1.08), sell all outcomes. Collect $1.08, pay out $1.00.

**Detection is pure math — no LLM needed:**
```
For each multi-outcome market:
  sum = Σ(all contract prices)
  if sum < 0.97:  → BUY ALL (spread = 1.00 - sum)
  if sum > 1.03:  → SELL ALL (spread = sum - 1.00)
  net_spread = spread - platform_fee_per_contract * n_contracts
  if net_spread > 0.02:  → EXECUTE
```

**Decision speed:** <500ms total (sum prices <1ms, fee calc <1ms, liquidity check ~50ms, parallel order placement ~200ms per contract).

---

## Module 2: Exotic Forex Triangular Arbitrage

### The Classic Triangle

```
USD → EUR → GBP → USD

USD $1000 → €920 (EUR/USD 1.087)
€920 → £790 (EUR/GBP 0.858)
£790 → $1,006 (GBP/USD 1.273)

Profit: $6 per loop. Run it 1000x.
```

**Math:** Multiply the three exchange rates. If product ≠ 1.000, arb exists. Further from 1.000 = fatter spread.

### Why Exotics Work

HFT firms dominate major pairs (EUR/USD, GBP/USD). But exotic crosses are underserved:

- **TRY, ZAR, BRL, MXN, HUF, CZK** — illiquid crosses misprice constantly
- **Capital controls** — official rate vs market rate divergence (Russia, Argentina, Turkey, Nigeria)
- **Geopolitical volatility** — every crisis creates pricing chaos across emerging market pairs
- **Onshore/offshore spread** — can differ 2-5% during crises

**Precedent:** Brad's friend retired at 40 arbing rubles during volatility periods. Onshore/offshore rate divergence + illiquid crosses = persistent mispricing.

### Exchange: Interactive Brokers (IBKR)

**IBKR is the primary exchange.** Reasons:
- 100+ forex pairs including all exotics (TRY, ZAR, MXN, HUF, CZK)
- Cheapest commissions (~$2 per $100K traded)
- REST + websocket API, Python SDK (`ib_insync`)
- 23 hours/day trading (forex market hours)
- Margin account for simultaneous leg execution
- **Won't ban you for arbing** — real broker, not market maker betting against you
- Paper trading mode for system validation before live capital

**Backup:** OANDA (70+ pairs, clean REST v20 API, simpler to prototype against)

**Avoid:** Robinhood (no exotics, no forex API), eToro (CFDs, bans arb), any "spread-only" broker (they're the market maker)

### Scanner Logic

```
For each permutation of 3 currencies from available pairs:
  product = rate_AB * rate_BC * rate_CA
  if abs(product - 1.0) > fee_threshold:
    → Calculate net profit after commissions
    → Check liquidity on all three legs
    → If profitable: ALERT (or auto-execute if confidence > threshold)
```

---

## Module 3: Crypto-Forex Bridge Arbitrage

### Opportunities

| Type | Detail |
|------|--------|
| **Kimchi/geographic premium** | BTC priced differently in different fiat currencies. Korea, Argentina, Nigeria premiums can hit 5-30%. |
| **Stablecoin-forex** | USDT trades at premiums in capital-control countries. Buy at $1, sell for $1.05 in pesos. |
| **CEX-CEX** | Same token, different prices on Binance vs Coinbase vs Kraken. Thin now but spikes during volatility. |
| **Funding rate arb** | Long spot + short perp when funding rate positive. Collect funding every 8 hours. Market-neutral yield, not speed-dependent. |
| **DEX forex pools** | Uniswap/Curve EUR/USD stablecoin pools misprice vs spot forex. |
| **Stablecoin depegs** | USDC/USDT/DAI occasionally trade at 0.998 or 1.003. Small spread, huge volume. |

### Exchanges

- **Kraken or Coinbase** for crypto leg (regulated, API access, fiat on/off ramps)
- **IBKR** for forex leg
- Agent watches price differential between BTC/USD on Kraken and implied rate through BTC/EUR → EUR/USD

---

## Module 4: Sports Betting Arbitrage

| Opportunity | Detail |
|------------|--------|
| **Cross-book arb** | DraftKings vs FanDuel vs BetMGM vs Caesars. Odds differ constantly. Alert when implied probabilities sum to < 100%. |
| **Line shopping** | Find best line across books for same bet. +EV, not pure arb. |
| **Live/in-play arb** | Odds move at different speeds during games. Fastest API wins. Google Fiber matters most here. |
| **Prop bet arb** | Player props have wide spreads across books (manually set). Most mispricing. |

**Aggregator:** The Odds API (theoddsapi.com) — 70+ books in one feed, $80/month.

**Risk:** Sportsbooks **will** limit/ban winning accounts eventually. Prediction markets don't do this (yet).

---

## Architecture

```
Kona (24/7, local, free compute — 128GB VRAM)
├── Polymarket MCP — websocket price feed
├── Kalshi MCP — websocket price feed
├── IBKR MCP — forex pair streaming
├── Odds API MCP — sportsbook aggregation
│
├── Arb Scanner (secretary tier — rules, no LLM)
│   ├── Cross-platform spread calculator
│   ├── Contract completeness checker (intra-platform)
│   ├── Triangular forex loop scanner
│   ├── Crypto-forex bridge monitor
│   └── Alert threshold: configurable per module
│
├── Slack alerts with: market, prices, spread %, liquidity depth
│
└── On alert → escalate to Thinker (Claude API)
    ├── Why does this spread exist? (news, settlement ambiguity, timing)
    ├── Liquidity check — can you actually fill at these prices?
    ├── Settlement risk — do both platforms resolve the same way?
    ├── Counterparty risk evaluation
    └── GO/NO-GO recommendation with position sizing
```

### Two-Tier Agent Model

**Secretary (Kona/Runtz — local, 24/7, free):**
- Continuous price monitoring across all feeds
- Pure math: addition, multiplication, comparison
- No LLM needed for detection — Python loops
- 4B-12B model only if natural language interpretation needed
- Decision speed: <500ms

**Thinker (Claude API — on-demand, paid per call):**
- Evaluates WHY a spread exists before execution
- Assesses liquidity depth, settlement risk, counterparty risk
- Position sizing recommendations
- Only invoked when secretary finds opportunity above threshold

### Latency Advantage

Google Fiber (symmetric gigabit, low jitter) provides edge over retail competitors on consumer internet. Not HFT-level, but prediction market spreads persist for minutes — more than enough. Most valuable for live sports arb where odds move fastest.

---

## Accounts Needed

| Service | Status | Action |
|---------|--------|--------|
| Polymarket | Not set up | Create account, fund with USDC |
| Kalshi | Not set up | Create account, KYC, fund |
| IBKR | Not set up | Create account, KYC, fund margin account |
| Robinhood | Existing? | Enable prediction markets |
| Kraken or Coinbase | Check | API keys for crypto leg |
| The Odds API | Not set up | $80/month subscription |
| Hyperspace | Not set up | `curl install`, no account needed |

---

## Module 5: Hyperspace AGI Network — Distributed Alpha Generation

### What It Is

[hyperspaceai/agi](https://github.com/hyperspaceai/agi) — a distributed P2P network of autonomous AI agents that collaboratively train models, share experiments via gossip protocol, and evolve strategies through mutation + selection. The commits in the repo are made by agents, not humans.

### AutoQuant (v2.6.9)

Hyperspace pointed Karpathy's autoresearch loop at quantitative finance. 135 autonomous agents evolved multi-factor trading strategies — mutating factor weights, position sizing, risk controls — backtesting against 10 years of market data, sharing discoveries via P2P gossip.

**What agents found (no human instruction):**
- Started: 8-factor equal-weight portfolios, Sharpe ~1.04
- Agents independently converged on dropping dividend, growth, and trend factors
- Switched to risk-parity position sizing
- Result: **Sharpe 1.32, 3x return, 5.5% max drawdown**
- Parsimony wins — fewer factors, better performance
- Cross-pollination via gossip accelerated convergence

### 5 Research Domains

| Domain | Metric | Relevance |
|--------|--------|-----------|
| Machine Learning | Validation loss | Model improvement |
| Search Engine | NDCG@10 | — |
| **Financial Analysis** | **Sharpe ratio** | **Direct alpha generation** |
| Skills & Tools | Test pass rate | Agent capability |
| Causes | Domain-specific | — |

### 9 Node Capabilities

Inference (GPU model serving), Research (ML training), Proxy (residential IP), Storage (DHT blocks), Embedding (CPU vectors), Memory (distributed vector store), Orchestration (multi-step task routing), Validation (proof verification), Relay (NAT traversal).

### How to Join

```bash
curl -fsSL https://agents.hyper.space/api/install | bash
hyperspace models pull --auto
hyperspace start
```

No API keys, no account registration, no port config. libp2p handles NAT traversal. Ed25519 for identity.

### GPU Model Recommendations

| VRAM | Model |
|------|-------|
| 4GB | Gemma 3 1B |
| 8GB | Gemma 3 4B |
| 16GB | Gemma 3 12B |
| 48GB | Gemma 3 27B |
| 80GB+ | Qwen2.5 Coder 32B |

### Our Fleet on the Network

| Machine | VRAM | Est. Points/Month |
|---------|------|-------------------|
| Kona | 128GB | 44K+ |
| Runtz | 128GB | 44K+ |
| Kush | 36GB | ~20K |

With 256GB+ VRAM we'd be one of the bigger nodes. Points earned + access to financial research discoveries from the entire network.

### How It Fits

| Component | Role |
|-----------|------|
| **Rust arb scanner** | Speed — detects mispricing in real-time (our build) |
| **Hyperspace agents** | Alpha — evolve trading strategies through collective research (their network) |
| **Claude thinker** | Judgment — evaluates opportunities before execution (on-demand) |

The arb scanner finds **risk-free math**. Hyperspace agents find **risk-adjusted alpha**. Different problems, complementary approaches. Run Hyperspace on idle compute cycles — when machines aren't scanning or doing FORGE inference, they contribute to the network and receive strategy insights.

### Why Now

"These are the small moments where money is made." — Brad

The network is early. 135 agents, growing. Being a large node early means:
- More influence on research direction
- Early access to discoveries via gossip
- Point accumulation before the network scales and dilutes
- Reputation as a serious node

---

## Module 6: AI Earnings Intelligence — The Original Thesis

### Concept

Real-time NLP inference on live earnings calls, trained on 10 years of transcripts per company. Detect sentiment shifts, guidance softening, hedging language, and executive tone changes BEFORE the market prices them. Trade the stock and — critically — the entire supply chain.

### Data Sources

- **10 years of earnings transcripts** — ~40K calls (S&P 500 × 40 quarters). Training data for per-CEO/CFO language baselines.
- **Live audio feed** — real-time inference during the call. ~4 calls/day during earnings season.
- **SEC filings** (10-K, 10-Q) — risk factors, segment data, guidance cross-reference.
- **Supply chain graph** — who sells to whom. Public data. Maps upstream suppliers, downstream customers, competitors.
- **Options flow** — pre-earnings positioning as a leading signal.

### Signal Types

| Signal | Example | Trade |
|--------|---------|-------|
| Hedging language increase | "We expect" → "We hope" | Short or put |
| Guidance softening | Subtle word choice shift | Fade the stock |
| Executive stress indicators | Tone/cadence changes (audio) | Directional bet |
| Question dodging | Analyst asks margins, CEO talks revenue | Short |
| New risk factor mentions | First mention of competitor/regulation | Sector rotation |
| Supply chain inference | NVDA beats → TSMC order flow up | Long upstream |
| Macro correlation | "AI spending acceleration" | Sector ETF long |

### The Macro Graph

One earnings call → 10+ tradeable signals across the supply chain:

```
Company earnings call (live)
├── Direct: stock move
├── Upstream: supplier inference (orders, inventory)
├── Downstream: customer inference (spending, adoption)
├── Competitors: market share language → pressure signals
└── Macro: sector ETFs, thematic plays
```

The market prices the reporting company in seconds. Second and third order effects on supply chain take HOURS TO DAYS. That's the window.

### Speed Advantage

- Generic traders: wait for transcript (30-60 min delay) or analyst summary (hours)
- Our system: inferring on sentence 3 of CEO's opening remarks
- 10 years of per-CEO baseline means "encouraged" from Satya ≠ "encouraged" from Pichai
- Live audio analysis catches tone that transcripts miss entirely

### Architecture (Research-Informed)

Based on S&P Global, LSEG, ECC Analyzer, and MarketSenseAI 2.0 research:

```
Earnings Call Pipeline:

1. INGEST: Live audio → Whisper/Parakeet STT → raw transcript
2. SEGMENT: Break transcript into business segments (not one blob)
3. EXTRACT: Hierarchical RAG extraction per segment
   ├── Paragraph-level signals (guidance, risk, outlook)
   ├── Fine-grained focus sentences (hedging, dodging, softening)
   └── Structured features (NOT vibes):
       - Guidance delta language score
       - Uncertainty marker count
       - Question-dodging index
       - Response evasiveness score
       - Sentiment deviation from CEO's 10-year baseline
       - Modal verb shifts ("will" → "may" → "hope")
       - Answer length vs analyst question complexity
       - New risk factor mentions (first-time topics)
4. COMPARE: Per-CEO language fingerprint (LLM embeddings across 10yr baseline)
   └── "Is this CEO more hedging than their personal normal?"
5. SCORE: Segment-level sentiment (not aggregate) → per-segment alpha signal
6. PROPAGATE: Supply chain graph
   ├── Upstream supplier inference
   ├── Downstream customer inference
   └── Competitor read-through
7. TRADE: 1-5 day post-call drift (NOT sentence-3 speed)
   ├── IBKR API for stock + options
   └── Position sizing based on signal confidence + liquidity
```

### Tech Stack

- **Whisper/Parakeet** — real-time speech-to-text on live audio (local, 128GB VRAM)
- **Frontier LLM** — Claude/GPT for structured feature extraction (NOT sentiment classification)
- **RAG pipeline** — hierarchical extraction per ECC Analyzer architecture
- **Embedding store** — per-CEO 10-year language baseline vectors (Qdrant)
- **Supply chain graph** — Neo4j, maps all public company relationships
- **Execution** — IBKR API for stock + options trades
- **Signal aggregation** — segment-level features + options flow + supply chain position
- **Backtesting** — walk-forward splits, conservative fills, look-ahead bias guard (pin model versions pre-earnings date)

---

## Build Order

1. **Prediction market intra-platform completeness arb** — simplest math, single platform, proves infrastructure
2. **Prediction market cross-platform arb** — adds Kalshi, cross-platform execution
3. **Exotic forex triangular arb** — IBKR integration, more complex execution
4. **Crypto-forex bridge** — connects crypto + forex legs
5. **Sports betting arb** — highest volume but account limiting risk
6. **Hyperspace node** — install on Kona/Runtz, join financial research network, earn points on idle cycles
7. **Earnings intelligence** — transcript corpus + live call inference + supply chain macro bets (the original thesis)

---

## Capital Requirements

### Phase 1: Prove the System ($5-10K)
- Paper trading on all modules for 30 days
- Live with small capital ($1-2K per module) for 30 days
- Track every arb detected, executed, and missed
- Prove the scanner catches real opportunities consistently

### Phase 2: Scale ($750K committed on proof)

| Strategy | Allocation | Conservative | Aggressive |
|----------|-----------|-------------|------------|
| Prediction market arb (all modes) | $200K | $40-60K/yr | $80-120K/yr |
| Forex triangular (exotics, IBKR margin) | $250K | $25-50K/yr | $75-125K/yr |
| Crypto yield + funding rate | $150K | $22-45K/yr | $45-75K/yr |
| Hyperspace alpha strategies | $100K | $15-25K/yr | $30-50K/yr |
| Crisis reserve (dry powder) | $50K | — | Deployed during fat spreads |
| **Total** | **$750K** | **$102-180K/yr** | **$230-370K/yr** |

The crisis reserve stays liquid. When geopolitical events (Iran, elections, black swans) blow spreads to 5-10x normal, deploy into the fattest opportunities. This is where outsized returns come from — having capital ready when everyone else is panicking.

### The Thesis

The scanner doesn't predict news. It's already running when news hits. Every quarter delivers a catalyst: geopolitical crisis, election cycle, crypto event, central bank surprise, new platform launch. The infrastructure cost is fixed (owned hardware). The news cycle is the product.

---

## Key Risks

- **Execution risk:** Prices move between detection and order fill (slippage)
- **Settlement risk:** Cross-platform — do both markets resolve the same event the same way?
- **Account risk:** Sportsbooks ban winners. Prediction markets don't (yet).
- **Regulatory risk:** Prediction market regulation still evolving in US. Willing to accept gray area risk at retail scale.
- **Liquidity risk:** Spread exists but order book too thin to fill at advertised price
- **Capital lock-up:** Prediction markets can take months to settle (election markets)
- **Legging risk:** One leg fills, other doesn't — leaves an unhedged directional position. Mitigate with position limits.
- **Platform risk:** Venues can freeze accounts, change fees, alter APIs, reject withdrawals. Don't concentrate capital on any single venue.
- **Catastrophic correlation:** In a true crisis, multiple modules may fail simultaneously. Hard daily loss limit across all modules.
- **Model drift:** AI models change behavior across versions. Pin model versions for production, test before upgrading.

## Mandatory Risk Infrastructure (Build Before Phase 2)

- [ ] Global kill switch — one command, all modules stop, all orders cancelled
- [ ] Per-module daily loss limit (configurable, default 2% of module allocation)
- [ ] Per-venue exposure cap (no more than 30% of total capital on any single venue)
- [ ] Stale data detection — exclude any price older than 60 seconds from arb calculations
- [ ] Duplicate order prevention
- [ ] Audit log for every trade (timestamp, venue, instrument, size, fill price, fees, slippage)
- [ ] PnL attribution waterfall: detected spread → submitted spread → filled spread → net after fees
- [ ] Tax lot tracking across all asset classes and venues
- [ ] Weekly reconciliation: expected vs actual P&L per module

---

## Build Sequence (Revised Post-Consortium)

Priority order based on: edge durability × AI leverage × retail advantage × capital efficiency

### Phase 1: Foundation (Weeks 1-4)
1. **Risk infrastructure** — kill switch, loss limits, audit logs, PnL waterfall tracker
2. **Module 1: Intra-platform completeness arb** — single venue (Polymarket), pure math, proves the scanner and execution pipeline. $1K max. Goal: measure detected vs captured edge over 90 days.

### Phase 2: Highest Ceiling (Weeks 5-12)
3. **Module 6: AI Earnings Intelligence** — start with ONE sector (semis: NVDA, AMD, INTC, TSM, ASML). 10 years of transcripts. Build per-CEO language baselines. Paper trade first, then $5K live. Don't race speed — race interpretation quality on 1-5 day post-call drift.

### Phase 3: Expand Proven Modules (Months 4-6)
4. **Module 1 expansion** — add Kalshi for cross-platform arb. Scale capital on proven module.
5. **Module 6 expansion** — add cloud (MSFT, GOOG, AMZN, META) and supply chain propagation.
6. **Module 3: Crypto yield** — funding rate arb only (market-neutral, no capital-control plays initially). $10K on top-tier venues with strict risk caps.

### Phase 4: Full Deployment (Months 7-12)
7. **Module 2: Forex triangular** — IBKR paper trading first, exotics only. Prove the triangle math survives fees and slippage before live capital.
8. **Module 4: Sports betting** — small scale, accept account mortality. Run until banned, rotate.
9. **Module 5: Hyperspace** — install on idle compute. Free option. Zero capital at risk.
10. **Phase 2 capital deployment ($750K)** — only after 6+ months of proven captured edge across multiple modules.

### Ongoing
- Track detected vs captured edge per module weekly
- Kill any module where captured edge is negative for 30 consecutive days
- Scale capital only to modules with proven positive captured edge
