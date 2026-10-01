# Matching Engine Console

A browser app around a price-time priority limit order book. It is a JavaScript port of the C++ `std::map` matching engine from my Stony Brook research on low-latency order books (Python vs C++, `std::vector` vs `std::map`).

**Live demo:** https://YOUR-USERNAME.github.io/matching-engine-console/

## Features
- Limit and market orders, partial fills, cancel for resting orders
- Live depth ladder with spread and mid, plus a trade tape
- Flow simulator with two models: uniform ±$5 around $100 (summer baseline) and a clustered, bursty model with a random-walk mid and fat-tailed price offsets
- 10,000-order benchmark that times each add call and reports mean, p50, p95, p99, max, throughput and price-level count

## Engine notes
- Prices are stored as integer cents, so price-level keys never suffer floating-point drift
- Each side keeps a sorted price array plus a map of price to FIFO queue, so the best price is always at index 0 and matching stops at the first non-crossing level
- Trades print at the resting order's price

## Run locally
Open `index.html` in a browser. There is no build step and no dependencies.

## Caveat
Browsers clamp `performance.now()` resolution, so microsecond latencies here are coarse. Compare the two flow models against each other, not against the C++ numbers.

## Roadmap
- WebSocket wrapper around the C++ `OrderBookMap` as a real backend
- Real crypto tick data replay
- CLOB vs AMM slippage panel
