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

---

## Build Order

1. **Prediction market intra-platform completeness arb** — simplest math, single platform, proves infrastructure
2. **Prediction market cross-platform arb** — adds Kalshi, cross-platform execution
3. **Exotic forex triangular arb** — IBKR integration, more complex execution
4. **Crypto-forex bridge** — connects crypto + forex legs
5. **Sports betting arb** — highest volume but account limiting risk

---

## Capital Requirements

- **Prediction markets:** Start with $1K to prove system, scale to $50K+
- **Forex:** $10K+ for meaningful margin account on IBKR
- **Crypto:** Variable, depends on exchange minimums
- **Total to start:** ~$5K to validate all modules

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
