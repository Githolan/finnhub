# Market Data Source Investigation Plan

This document captures the actionable plan to understand and document how market snapshots and deep analysis data choose providers (Finnhub vs. Binance), including symbol conversion rules and fallbacks.

## 1. Trace snapshot and deep-analysis pipelines
- Locate API route definitions for:
  - `GET /api/v1/market/snapshot`
  - `POST /api/v1/analysis/deep`
- Follow each handler into the service layer (e.g., `fetch_market_snapshot`, `fetch_deep_analysis_data`).
- Identify how exchange/market type is parsed from requests and propagated.
- Record any existing fallback logic or provider selection branching.

## 2. Map symbol normalization per provider
- Search utilities/helpers that transform user-entered symbols into provider-specific tickers.
- Capture rules for:
  - Finnhub equities (e.g., `AAPL`, `MSFT`)
  - Finnhub crypto (prefixed forms like `BINANCE:BTCUSDT` if applicable)
  - Binance spot API (e.g., `BTCUSDT`, `ETHUSDT`), including case handling.
- Note how explicit exchange input (e.g., `BINANCE`, `NASDAQ`) influences conversion and validation.

## 3. Define preferred source matrix and fallback rules
- For each market category (US/EU equities, crypto, others), document intended primary provider and fallback order.
- Verify each provider exposes required fields for MarketSnapshot (price, change %, day range, volume).
- Note timeout/error handling that triggers fallbacks and whether partial data is acceptable.

## 4. Confirm MarketSnapshot response contract
- Identify DTO/serializer that shapes the response returned to the Front End.
- Ensure field names, units, rounding, and day range conventions match FE expectations.
- Check caching layers (if any) that may affect freshness or fallback behavior.

## 5. Audit deep analysis data assembly
- Trace `fetch_deep_analysis_data(symbol, timeframe, exchange)` through indicator modules (e.g., Fibonacci, Awesome Oscillator, Alligator, trend recognition).
- Document which provider supplies OHLCV per timeframe and whether symbol conversion differs from snapshots.
- Determine error propagation: when does the endpoint return 4xx vs 5xx, and are partial indicator results allowed?

## 6. Validate end-to-end with representative scenarios
- Prepare test cases for:
  - US stock (e.g., `AAPL` on `NASDAQ`)
  - EU stock (if supported)
  - Crypto (`BTCUSDT` on `BINANCE`, `BTC/USD` variants)
  - Edge cases: unknown exchange, lowercase symbols, malformed input
- Exercise both endpoints locally (or via tests) to observe provider selection, symbol conversion, and fallback.
- Capture discrepancies between observed behavior and the desired source priority matrix.

## 7. Deliverables
- A source-selection matrix documenting preferred providers and fallbacks per market.
- Symbol conversion cheat-sheet for Finnhub and Binance.
- Sequence diagrams (updated if necessary) showing provider selection and fallback transitions.
- Issues/PRs created for any gaps discovered during investigation.
