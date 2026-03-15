# Wall Street Scanner — Architecture

## Language Decision

**Rust for the scanner core. Python for the MCP bridge.**

Why Rust:
- 24/7 process on Kona — needs to run forever without memory leaks or GC pauses
- Sub-500ms detection-to-alert on arb opportunities
- Websocket multiplexing across 100+ feeds with zero overhead
- Money on the line — Rust's type system catches bugs at compile time, not at 2am
- Single binary, ~5MB, ~20MB RAM. No virtualenv, no pip, no dependency rot.

Why not Python:
- GIL limits true concurrency across feeds
- GC pauses introduce latency jitter
- Python daemons bloat over time, need babysitting
- asyncio is fine for MCP servers, wrong for a trading scanner

## Structure

```
wallstreet/
├── scanner/                  # Rust crate — the Porsche
│   ├── Cargo.toml
│   ├── src/
│   │   ├── main.rs           # Tokio async runtime, CLI, startup
│   │   ├── config.rs         # TOML config loading
│   │   ├── types.rs          # Price, Market, Opportunity, ForexPair
│   │   ├── error.rs          # thiserror enum
│   │   ├── feeds/
│   │   │   ├── mod.rs        # PriceFeed trait
│   │   │   ├── polymarket.rs # Polygon CLOB websocket
│   │   │   ├── kalshi.rs     # Kalshi websocket
│   │   │   └── ibkr.rs       # IBKR TWS API (forex)
│   │   ├── arb/
│   │   │   ├── mod.rs
│   │   │   ├── completeness.rs  # Sum != $1.00 (intra-platform)
│   │   │   ├── cross.rs         # Cross-platform spread
│   │   │   └── triangle.rs     # Forex triangular loops
│   │   └── alert.rs          # Slack webhook + tracing
│   └── config/
│       └── scanner.toml      # Default config
├── mcp-bridge/               # Python — thin MCP wrapper
│   └── server.py             # Exposes scanner state as MCP tools
├── STRATEGY.md               # Market analysis & build order
└── ARCHITECTURE.md           # This file
```

## Key Crates

- `tokio` — async runtime (full features)
- `tokio-tungstenite` — websocket client
- `rust_decimal` — NEVER use f64 for money. Ever.
- `dashmap` — concurrent hashmap for shared price state across feeds
- `serde` + `serde_json` — JSON parsing
- `reqwest` + rustls — HTTP (REST APIs, Slack alerts)
- `tracing` — structured logging
- `clap` — CLI args
- `chrono` — timestamps

## Core Design

### Shared State
```
Arc<DashMap<String, Market>>         — prediction market prices
Arc<DashMap<(String,String), ForexPair>>  — forex pair prices
```

Each feed runs as a tokio task, writes to shared state via DashMap (lock-free concurrent reads).

### Scanner Loop
```
loop {
    sleep(scan_interval)  // 100ms default

    // Prediction markets
    for market in prediction_markets:
        completeness::check(&market)     // sum != $1.00?
        cross::check(&market, &other)    // same event, different price?

    // Forex
    triangle::scan(&forex_pairs)         // all 3-currency permutations

    // Alert on any opportunities found
    for opp in opportunities:
        if cooldown_ok(&opp):
            alert::send(&opp)            // Slack + tracing
}
```

### Feed Trait
```rust
#[async_trait]
trait PriceFeed: Send + Sync {
    async fn connect(&mut self) -> Result<()>;
    async fn subscribe(&self, markets: &[String]) -> Result<()>;
    fn platform(&self) -> Platform;
}
```

Auto-reconnect with exponential backoff on disconnect. Stale price detection (>60s without update = exclude from arb calc).

### Money Rules
- All prices: `rust_decimal::Decimal`. No exceptions.
- Fees subtracted BEFORE alerting. Net profit must be positive after all fees.
- Forex triangle uses ask for buy legs, bid for sell legs (realistic execution prices).
- Liquidity check: opportunity confidence based on order book depth.

## MCP Bridge

Thin Python FastMCP server that reads scanner state (via Unix socket or HTTP) and exposes:
- `scanner_status` — what feeds are connected, last update times
- `active_opportunities` — current open arb opportunities
- `scanner_history` — recent alerts with P&L tracking

This lets any FORGE agent ask "what arbs are open right now?" without touching Rust.

## Runtime

- **Kona** (128GB VRAM Mac Mini M4 Max) — primary host, 24/7
- **Google Fiber** — symmetric gigabit, low jitter
- Single binary, ~5MB on disk, ~20MB RAM
- Managed by launchd (same pattern as all FORGE services)

## Build Order

1. Types + config + error handling (the foundation)
2. Completeness arb detection (pure math, testable without live feeds)
3. Polymarket feed (first live data source)
4. Cross-platform detection (needs two feeds)
5. Kalshi feed + IBKR feed
6. Triangle forex scanner
7. Alert system (Slack)
8. MCP bridge
9. Paper trading validation
10. Live execution

## Blocked On

- [ ] IBKR account setup
- [ ] Polymarket account + USDC funding
- [ ] Kalshi account + KYC
- [ ] Kona hardware arrival
- [ ] Rust toolchain on Kona (`rustup`)
