# Wall Street Agent — Strategy & Architecture

## Overview

Automated arbitrage detection and execution using secretary-tier agents running 24/7 on local hardware (Kona/Runtz). Thinker-tier (Claude API) evaluates trade quality before execution. Google Fiber provides latency advantage over retail competitors.

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

### Tech Stack

- **Whisper/Parakeet** — real-time speech-to-text on live audio
- **Fine-tuned LLM** — trained on 10 years of per-CEO language patterns
- **Supply chain graph** — Neo4j or similar, maps all public company relationships
- **Execution** — IBKR API for stock + options trades
- **Signal aggregation** — combine transcript signal + options flow + supply chain position

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
- **Regulatory risk:** Prediction market regulation still evolving in US
- **Liquidity risk:** Spread exists but order book too thin to fill at advertised price
- **Capital lock-up:** Prediction markets can take months to settle (election markets)

---

## Timeline

Blocked on hardware (Kona/Runtz arrival). When iron arrives:
1. Week 1: Polymarket MCP server + intra-platform scanner
2. Week 2: Kalshi MCP server + cross-platform scanner
3. Week 3: IBKR account setup + forex MCP server
4. Week 4: Full system integration + paper trading validation
