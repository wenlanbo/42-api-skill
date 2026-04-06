---
name: 42-api
description: Query 42.space prediction market data via REST API. Use when asked about 42 markets, outcome token prices, trader positions, leaderboards, OHLC charts, market stats, or any 42.space platform data. Covers market discovery, pricing, portfolio tracking, and trader analytics. No API key required. Works with any HTTP client (curl, fetch, requests, etc.).
---

# 42 REST API

Base URL: `https://rest.ft.42.space`

All endpoints are **read-only GET requests** — no authentication, no API key, no headers required. Standard JSON responses with `{data, pagination}` for lists. Use any HTTP client: `curl`, Python `requests`, JS `fetch`, etc.

## Quick Reference

### Markets

```bash
# List live markets sorted by volume
curl -s "https://rest.ft.42.space/api/v1/markets?status=live&order=volume&limit=10"

# Get single market by address
curl -s "https://rest.ft.42.space/api/v1/markets/{address}"

# Get market lifecycle events
curl -s "https://rest.ft.42.space/api/v1/markets/{address}/timeline"

# List categories
curl -s "https://rest.ft.42.space/api/v1/markets/categories"

# List all outcome tokens (filterable by market)
curl -s "https://rest.ft.42.space/api/v1/markets/tokens?market_address={address}"
```

### Pricing & Charts

```bash
# Current prices for all outcomes in a market
curl -s "https://rest.ft.42.space/api/v1/market-data/prices?market={address}"

# Price history (durations: 1h, 4h, 24h, 7d, 30d, 90d, 1y, all)
curl -s "https://rest.ft.42.space/api/v1/market-data/prices/history?market={address}&outcome_index=0&duration=7d"

# OHLC candles (intervals: 10s, 1m, 3m, 30m, 2h, 6h, 12h, 1d)
curl -s "https://rest.ft.42.space/api/v1/market-data/ohlc?market={address}&outcome_index=0&interval=1d"
```

### Stats

```bash
# Batch stats for up to 50 markets
curl -s "https://rest.ft.42.space/api/v1/market-data/stats?market={addr1},{addr2}"

# Outcome token stats with price/volume/collateral deltas (30m, 1h, 4h, 24h)
curl -s "https://rest.ft.42.space/api/v1/market-data/tokens/stats?status=live&order_by=volume"

# Token holders for a specific outcome
curl -s "https://rest.ft.42.space/api/v1/market-data/holders?market={address}&token_id={id}"
```

### User / Portfolio

```bash
# Open positions (includes unrealised PnL)
curl -s "https://rest.ft.42.space/api/v1/market-data/positions?user={wallet}"

# Closed positions (includes realised PnL, win/loss)
curl -s "https://rest.ft.42.space/api/v1/market-data/closed-positions?user={wallet}"

# Activity feed (MINT, REDEEM, FINALISE, CLAIM)
curl -s "https://rest.ft.42.space/api/v1/market-data/activity?user={wallet}"
```

### Leaderboard

```bash
# Global trader rankings (sort: pnl, win_rate, volume; period: 1d, 7d, 30d, all)
curl -s "https://rest.ft.42.space/api/v1/market-data/leaderboard?sort_by=pnl&period=7d&limit=20"

# Single wallet stats (lifetime PnL, win rate, best/worst trade)
curl -s "https://rest.ft.42.space/api/v1/market-data/leaderboard/stats/{wallet}"

# Top trades globally
curl -s "https://rest.ft.42.space/api/v1/market-data/leaderboard/top-trades?period=7d&limit=10"
```

## Pagination

All list endpoints accept `limit` and `offset`. Response includes:
```json
{"data": [...], "pagination": {"hasMore": true, "totalResults": 150}}
```

Default limits vary per endpoint (10–100). Max is typically 500 (50 for closed positions, 100 for holders).

## Key Filters

**Markets:** `status` (live|ended|resolved|finalised|all), `category`, `tag`, `collateral`, `volume_min`/`volume_max`, date ranges
**Token stats:** `order_by` (volume|price|collateral|traders), `category`, `status`, `market_address`
**Activity:** `type` (MINT|REDEEM|FINALISE|CLAIM), time range via `start`/`end` unix timestamps
**Leaderboard:** `period` (1d|7d|30d|all), `sort_by` (pnl|win_rate|volume)

## Response Schemas

For full OpenAPI schemas per endpoint, see [references/api-schemas.md](references/api-schemas.md).

## 42 Market Model

- Markets have **multiple outcome tokens** (not binary YES/NO like Polymarket)
- **Parimutuel settlement** — losers' pool redistributed to winners pro-rata
- Outcome tokens trade on a **bonding curve** with continuous pricing
- Collateral is typically USDT on BSC (BNB Chain)
- Each outcome has: `price`, `volume`, `marketCap`, `payout`, `mintedQuantity`
- Market statuses: `live` → `ended` → `resolved` → `finalised`

## Common Workflows

**Find a market:** Search `/markets` by category, tag, or status → get `address` → use it in all other endpoints.

**Track a portfolio:** `/positions?user={wallet}` for open, `/closed-positions?user={wallet}` for history, `/activity?user={wallet}` for trade-by-trade feed.

**Monitor price movement:** `/tokens/stats` gives 30m/1h/4h/24h deltas across all live outcomes — ideal for alerts and dashboards.

**Build a leaderboard view:** `/leaderboard` for rankings, `/leaderboard/stats/{wallet}` for deep wallet analytics, `/leaderboard/top-trades` for highlights.
