# Matching Engine Console

A browser app around a price-time priority limit order book. It is a JavaScript port of the C++ `std::map` matching engine from my Stony Brook research on low-latency order books (Python vs C++, `std::vector` vs `std::map`), extended for a Fall 2026 study of how the engine behaves on real crypto trade data.

**Live demo:** https://seoeun-24.github.io/matching-engine-console/

## Features
- Limit and market orders, partial fills, and cancel for resting orders
- Live depth ladder with spread and mid, plus a trade tape
- Flow simulator with two synthetic models: uniform ±$5 around $100 (the summer baseline) and a clustered, bursty model with a random-walk mid and fat-tailed price offsets
- Benchmark that reports mean, p50, p95, p99, max, throughput, trade count and price-level count
- **Real data replay:** load one or more CSV files of trades and run each through a fresh engine
- **Day-by-day comparison:** select several files at once (named by date) and get one table row per file, a "vs first" column showing throughput relative to the first file, and a summary naming the fastest and slowest day

## Using your own data
Upload CSV files with a header row and these columns:

```
timestamp,price,qty,side
1759276800204,59999.55,0.16,buy
1759276800473,59999.68,0.20,sell
```

- `price` and `qty` are required. `side` (buy or sell) is optional; without it the side is inferred with a tick rule (an up-tick counts as a buy).
- Binance `is_buyer_maker` columns are understood. Raw Binance spot files have no header, so add one: `agg_trade_id,price,qty,first_trade_id,last_trade_id,timestamp,is_buyer_maker`.
- `timestamp` is kept for future use and is not used yet.
- Keep files to roughly 50k to 100k rows each, since everything runs in the browser.
- Prices are scaled to integer cents and quantities are multiplied by 1000 and rounded.
- To compare days, name files by date (for example `2026-09-28.csv`), select them all together, then click the benchmark button.

`test_trades.csv` in this repo is synthetic data for checking the loader. It is not exchange data.

## Engine notes
- Prices are stored as integer cents, so price-level keys never suffer floating-point drift
- Each side keeps a sorted price array plus a map of price to FIFO queue, so the best price is always at index 0 and matching stops at the first non-crossing level
- Trades print at the resting order's price

## Run locally
Open `index.html` in a browser. There is no build step and no dependencies.

## Limitations
- **Timing:** browsers round `performance.now()` to about 100 microseconds, so single calls cannot be timed. The mean comes from total time. The p50, p95, p99 and max columns are percentiles of 250-order batch averages, so they understate true per-order tails. Compare runs against each other, not against the C++ numbers.
- **Replay is an approximation:** executed trades are replayed as limit orders at the trade price. Cancelled and never-filled orders are not in trade data, so full order book feeds would be more faithful.
- **Visible book:** the ladder and trade tape show manual orders and the live flow simulator. CSV data only feeds the benchmark table.

## Roadmap
- Compile the C++ `OrderBookMap` to WebAssembly so latency is measured in C++ with `std::chrono`
- Replay a loaded file through the visible order book at adjustable speed
- CLOB vs AMM slippage panel
