# 42 API Skill

Query [42.space](https://42.space) prediction market data from any AI agent. No API key required.

## What is 42?

42 is an onchain prediction market on BSC (BNB Chain) where users trade multi-outcome tokens on real-world events. Unlike binary YES/NO markets, 42 markets have multiple outcomes with parimutuel settlement — losers' pool is redistributed to winners pro-rata.

Backed by YZi Labs (prev. Binance Labs), DWF Labs, and others.

- **Docs:** https://docs.42.space
- **App:** https://42.space

## What This Skill Does

Gives your AI agent full read access to 42's REST API — **17 endpoints** covering:

| Section | Endpoints | What You Get |
|---------|-----------|--------------|
| **Markets** | 6 | Discovery, filtering, categories, outcome tokens, timeline |
| **Pricing** | 3 | Current prices, historical series, OHLC candlesticks |
| **Stats** | 3 | Batch market stats, token-level deltas, holder lists |
| **Portfolio** | 3 | Open/closed positions, activity feed by wallet |
| **Leaderboard** | 3 | Trader rankings, wallet analytics, top trades |

## Quick Start

Drop the `42-api/` folder into your agent's skills directory. The agent will automatically use it when asked about 42 markets, prices, or trader data.

### Example Prompts

- *"Show me the top 5 live markets on 42 by volume"*
- *"What are the current outcome prices for the Polymarket FDV market?"*
- *"Check my wallet's open positions: 0xABC..."*
- *"Who are the top traders on 42 this week?"*
- *"Get the 7-day price history for outcome 0 on market 0x4E44..."*

### Example API Call

```bash
# No auth needed — just hit the endpoint
curl -s "https://rest.ft.42.space/api/v1/markets?status=live&order=volume&limit=5"
```

## Compatibility

Works with any AI agent that can read markdown and make HTTP requests:

- ✅ Claude Code / Claude Desktop
- ✅ OpenAI Codex
- ✅ Cursor
- ✅ Windsurf
- ✅ OpenClaw / Clawdbot
- ✅ Any MCP-compatible agent
- ✅ Custom agents with shell/HTTP access

## Repository Structure

```
.
├── README.md                          # This file
├── LICENSE                            # MIT License
└── 42-api/                            # Skill folder — drop this into your agent
    ├── SKILL.md                       # Main skill file (quick reference + curl examples)
    └── references/
        └── api-schemas.md             # Full endpoint docs (parameters + response schemas)
```

**To install:** Copy the `42-api/` directory into your agent's skills folder. That's it.

- **SKILL.md** (~5KB): Concise quick-reference with copy-paste curl examples for all 17 endpoints, plus common workflow patterns
- **api-schemas.md** (~10KB): Complete parameter tables and response field documentation for every endpoint

## API Overview

**Base URL:** `https://rest.ft.42.space`

All endpoints are read-only `GET` requests. No authentication, no API keys, no special headers. Standard JSON responses.

### Key Endpoints

```bash
# Markets
GET /api/v1/markets                              # List/filter markets
GET /api/v1/markets/{address}                     # Single market details
GET /api/v1/markets/{address}/timeline            # Lifecycle events
GET /api/v1/markets/categories                    # All categories
GET /api/v1/markets/categories/{slug}             # Category by slug
GET /api/v1/markets/tokens                        # All outcome tokens

# Pricing & Charts
GET /api/v1/market-data/prices                    # Current outcome prices
GET /api/v1/market-data/prices/history            # Price history
GET /api/v1/market-data/ohlc                      # OHLC candlesticks

# Stats
GET /api/v1/market-data/stats                     # Batch market stats (up to 50)
GET /api/v1/market-data/tokens/stats              # Token stats with deltas
GET /api/v1/market-data/holders                   # Outcome token holders

# Portfolio
GET /api/v1/market-data/positions                 # Open positions by wallet
GET /api/v1/market-data/closed-positions          # Closed positions
GET /api/v1/market-data/activity                  # User activity feed

# Leaderboard
GET /api/v1/market-data/leaderboard               # Trader rankings
GET /api/v1/market-data/leaderboard/stats/{wallet} # Wallet deep stats
GET /api/v1/market-data/leaderboard/top-trades    # Top trades globally
```

## 42 Market Model

| Concept | Description |
|---------|-------------|
| **Multi-outcome** | Markets have 2+ outcomes (not binary YES/NO) |
| **Bonding curve** | Continuous pricing — no order book needed |
| **Parimutuel** | Winners split the losers' pool pro-rata |
| **Collateral** | USDT on BSC |
| **Lifecycle** | `live` → `ended` → `resolved` → `finalised` |

## Links

- **42 Docs:** https://docs.42.space
- **API Base:** https://rest.ft.42.space
- **42 App:** https://42.space
- **Twitter:** https://x.com/42space

## License

MIT
