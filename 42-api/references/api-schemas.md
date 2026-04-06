# 42 API — Full Endpoint Reference

Base: `https://rest.ft.42.space`

## Table of Contents

1. [Markets](#markets)
2. [Pricing & Charts](#pricing--charts)
3. [Stats](#stats)
4. [User / Portfolio](#user--portfolio)
5. [Leaderboard](#leaderboard)

---

## Markets

### GET /api/v1/markets — Get all markets

Paginated list with rich filtering.

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `limit` | int | Page size (default 100, max 500) |
| `offset` | int | Offset (default 0) |
| `order` | string | Sort: `created_at`, `volume`, `collateral`, `start_timestamp` |
| `ascending` | bool | Sort direction (default false) |
| `status` | string | `live`, `ended`, `resolved`, `finalised`, `all` |
| `category` | string | Category name (case-insensitive) |
| `category_id` | int | Category ID |
| `tag` | string | Tag slug |
| `collateral` | string | Collateral symbol or address |
| `volume_min` / `volume_max` | number | Volume range |
| `start_date_min` / `start_date_max` | string | ISO 8601 start date range |
| `end_date_min` / `end_date_max` | string | ISO 8601 end date range |
| `question_id` | string | Comma-separated question IDs |
| `market_address` | string | Comma-separated market addresses |

**Response — MarketResponse:**
```
address, questionId, question, slug, collateralAddress, collateralSymbol,
collateralDecimals, curve, startDate, endDate, status, finalisedAt, elapsedPct,
image, description, resolvedAnswer, totalMarketCap, volume, traders,
categories[], subcategories[], tags[], topics[],
proposer: {name, image, id, createdAt},
outcomes[]: {tokenId, name, index, price, volume, marketCap, payout,
             mintedQuantity, symbol, image, holdersAtFinalisation}
```

### GET /api/v1/markets/{market_address} — Get market by address

Single market. Same MarketResponse schema.

### GET /api/v1/markets/{market_address}/timeline — Get market timeline

Lifecycle events for the question backing this market.

**Response — TimelineEvent[]:**
```
type, timestamp, answer, newEndTimestamp,
winningOutcomes[]: {name, symbol}
```

### GET /api/v1/markets/categories — Get all categories

Returns all market categories.

### GET /api/v1/markets/categories/{slug} — Get category by slug

Single category by slug.

### GET /api/v1/markets/tokens — Get all outcome tokens

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `market_address` | string | Filter by market |
| `token_id` | string | Filter by token ID |
| `outcome_index` | int | Filter by index (0-based) |
| `limit` | int | Default 50, max 500 |
| `offset` | int | Default 0 |
| `order` | string | `price`, `market_cap`, `token_id`, `outcome_index` |
| `ascending` | bool | Default false |

**Response — TokenResponse:**
```
tokenId, outcomeName, outcomeIndex, price, marketCap,
market: {marketAddress, slug, question: {questionId, slug, title}}
```

---

## Pricing & Charts

### GET /api/v1/market-data/prices — Current outcome token prices

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `market` | string | **Required.** Market address |
| `token_id` | string | Single token ID |
| `outcome_index` | int | Single index (0-based) |
| `token_ids` | string | Comma-separated batch (max 50) |
| `outcome_indices` | string | Comma-separated batch (max 50) |
| `projection` | string | `full` (default) or `midpoint` |

**Response — PriceResponse[]:**
```
market, tokenId, outcome, price, mintedQuantity, outcomeCap,
totalMarketCap, updatedAt
```

### GET /api/v1/market-data/prices/history — Price history

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `market` | string | **Required.** Market address |
| `token_id` | string | Token ID (or use outcome_index) |
| `outcome_index` | int | 0-based (or use token_id) |
| `duration` | string | `1h`, `4h`, `24h`, `7d`, `30d`, `90d`, `1y`, `all` |
| `start_ts` / `end_ts` | int | Unix timestamps |
| `fidelity` | int | Max data points (downsamples if exceeded) |

**Response:**
```
outcomeName,
history[]: {t (unix), p (price), v (volume), payout}
```

### GET /api/v1/market-data/ohlc — OHLC candlesticks

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `market` | string | **Required.** Market address |
| `token_id` | string | Token ID (or use outcome_index) |
| `outcome_index` | int | 0-based (or use token_id) |
| `interval` | string | **Required.** `10s`, `1m`, `3m`, `30m`, `2h`, `6h`, `12h`, `1d` |
| `start_ts` / `end_ts` | int | Unix timestamps |
| `limit` | int | Max data points (default 500, max 5000) |

**Response — OHLCDataPoint[]:**
```
t (unix), o (open), h (high), l (low), c (close), v (volume),
payoutOpen, payoutHigh, payoutLow, payoutClose, payoutMid
```

---

## Stats

### GET /api/v1/market-data/stats — Batch market stats

Lightweight polling stats for up to 50 markets.

**Parameters:** `market` (required, comma-separated addresses, max 50)

**Response — MarketStatsResponse[]:**
```
market, marketCap, totalVolume, traders, updatedAt,
outcomeStats[]: {tokenId, name, price, marketCap, payout}
```

### GET /api/v1/market-data/tokens/stats — Outcome token stats

Per-outcome stats with time-bucketed deltas.

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `limit` | int | Default 100, max 500 |
| `offset` | int | Default 0 |
| `order_by` | string | `volume`, `price`, `collateral`, `traders` |
| `ascending` | bool | Default false |
| `category` / `category_id` | string/int | Filter |
| `status` | string | `live`, `ended`, `resolved`, `finalised`, `all` |
| `market_address` | string | Comma-separated |

**Response — TokenStatsEntry:**
```
tokenId, marketAddress, questionId, price, payout, collateral,
totalVolume, buyVolume, sellVolume, traders,
outcome: {name, symbol, image},
question: {title, description, image, endDate, categoryId},
market: {collateralDecimals},
statsChanges: {
  priceChange30m, priceChange1h, priceChange4h, priceChange24h,
  volumeChange30m, volumeChange1h, volumeChange4h, volumeChange24h,
  collateralChange30m, collateralChange1h, collateralChange4h, collateralChange24h
}
```

### GET /api/v1/market-data/holders — Outcome token holders

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `market` | string | **Required.** Market address |
| `token_id` | string | **Required.** Outcome token ID |
| `limit` | int | Default 20, max 100 |
| `offset` | int | Default 0 |

**Response — HolderEntry[]:**
```
userAddress, amount, heldQuantity, mintedQuantity,
avgPrice, currentPrice, payout, realizedPnl, unrealizedPnl,
outcomeName, outcomeSymbol
```

---

## User / Portfolio

### GET /api/v1/market-data/positions — Open positions

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `user` | string | **Required.** Wallet address |
| `market` | string | Comma-separated market addresses |
| `question_id` | string | Comma-separated question IDs |
| `limit` | int | Default 100, max 500 |
| `offset` | int | Default 0, max 10000 |

**Response — PositionResponse:**
```
tokenId, userAddress, marketAddress, questionId,
size, avgPrice, curPrice, costBasis, currentValue,
cashPnl, realizedPnl, percentPnl,
isClaimed, isFinalized, isWinner,
outcome: {index, name, symbol, image, mintedQuantity, payout},
question: {questionId, title, description, image, endDate, resolvedAnswer, winningOutcomes},
market: {collateralDecimals, startDate}
```

### GET /api/v1/market-data/closed-positions — Closed positions

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `user` | string | **Required.** Wallet address |
| `market` | string | Comma-separated |
| `question_id` | string | Comma-separated |
| `limit` | int | Default 10, max 50 |
| `offset` | int | Default 0, max 100000 |

**Response — ClosedPositionResponse:**
```
tokenId, userAddress, marketAddress, questionId,
size, avgPrice, curPrice, costBasis, realizedPnl, closedAt,
isClaimed, isFinalized, isWinner,
outcome: {index, name, symbol, image, mintedQuantity, payout},
question: {questionId, title, description, image, endDate, resolvedAnswer, winningOutcomes},
market: {collateralDecimals, startDate}
```

### GET /api/v1/market-data/activity — User activity feed

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `user` | string | Wallet address |
| `market` | string | Comma-separated market addresses |
| `question_id` | string | Comma-separated question IDs |
| `type` | string | `MINT`, `REDEEM`, `FINALISE`, `CLAIM` |
| `start` / `end` | int | Unix timestamp range |
| `limit` | int | Default 100, max 500 |
| `offset` | int | Default 0, max 10000 |

**Response — ActivityResponse:**
```
type, userAddress, marketAddress, questionId, tokenId,
title, outcome, outcomeSymbol, outcomeImage, outcomeIndex, questionImage,
size, price, tradePrice, avgPrice, collateral, currentQuantity,
marketCapAtTime, payoutAtTime, realizedPnlDelta, transactionHash, timestamp
```

---

## Leaderboard

### GET /api/v1/market-data/leaderboard — Trader leaderboard

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `period` | string | `1d`, `7d`, `30d`, `all` (default `all`) |
| `sort_by` | string | `pnl`, `win_rate`, `volume` (default `pnl`) |
| `user` | string | Wallet address (returns single rank) |
| `limit` | int | Default 100, max 500 |
| `offset` | int | Default 0 |

**Response — LeaderboardEntry:**
```
rank, walletAddress, pnl, pnlPct, winRate,
totalVolume, tradeCount,
winningPredictions, losingPredictions,
profitableSells, losingSells
```

### GET /api/v1/market-data/leaderboard/stats/{wallet_address} — Wallet trading stats

Lifetime aggregate stats for one wallet.

**Response — WalletStats:**
```
walletAddress, pnl, winRate, predictionWinRate, sellWinRate,
totalVolume, buyVolume, sellVolume, tradeCount, avgTradeSize,
winningPredictions, losingPredictions, profitableSells, losingSells,
bestTrade, worstTrade
```

### GET /api/v1/market-data/leaderboard/top-trades — Top trades

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| `period` | string | `1d`, `7d`, `30d`, `all` (default `all`) |
| `limit` | int | Default 10, max 50 |
| `offset` | int | Default 0 |

**Response — TopTradeEntry:**
```
rank, walletAddress, pnl, pnlPct, avgPrice,
deltaCollateral, deltaQuantity, eventType,
questionTitle, questionImage, outcomeName, outcomeImage, resolvedAt
```
