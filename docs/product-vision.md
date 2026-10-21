# Product Vision

## What FlowRadar is

FlowRadar is a web platform that shows order flow in real time and finds trading opportunities
with AI. It aims at the level of Coinalyze or Exocharts, with an AI layer on top.

It does two jobs:

1. **Chart the market.** It draws candles, footprint, TPO or volume profile with POC, VAH and
   VAL, CVD, open interest, funding, and liquidation heatmaps.
2. **Find setups.** It scans the market and publishes trade setups. Each setup has a trigger, an
   invalidation, an entry range, a stop loss, take-profit targets, R:R and a confidence score.
   Clicking a setup draws green and red Entry, Stop Loss and Take Profit boxes on the chart.

## Principle: reuse, do not rebuild

No single open-source project covers both order flow charting and an AI radar. Four projects
cover about 80% of the data and chart work (see [open-source-research.md](open-source-research.md)).
FlowRadar takes their code and patterns and spends its own effort on the part nobody has: the
Opportunity Radar and the accuracy pipeline behind it.

## Layout

The workspace is split in two:

- Left, 70% width: the chart (K-line, TPO profile, CVD, OI and funding sub-panes).
- Right, 30% width: the live Opportunity Radar feed, filterable by Intraday (scalp and day) or
  HTF (swing).

## Key features

- **Setups drawn on the chart.** A setup is not a text box. It becomes boxes on the chart.
- **Event-driven AI calls.** The AI runs on a candle close or on an order flow anomaly, not on a
  blind timer. This cuts token cost by roughly 70%.
- **Live setup tracking.** Every setup has a lifecycle. A setup that price has already passed is
  marked invalidated automatically, so the feed has no zombie signals.
- **Deterministic guardrails.** The AI never invents prices or does the math. Code does.

## Scope

| In the MVP | Out of the MVP |
|---|---|
| Public read-only chart and radar feed | Login, accounts, paid plans (D4) |
| Signals only (D1) | Placing orders, exchange API keys, position sizing |
| Crypto perpetuals on Binance and Bybit, derivatives data from CoinGlass | Other asset classes, other venues |
| Win rate and invalidation rate on setups (D2) | Per-user performance |

The research report also describes an execution and risk layer (bracket orders, MCP exchange
servers, dry run). That layer is deferred. See D1 and [known-issues.md](known-issues.md).

## External costs

- CoinGlass Standard API: about $299 per month, fixed. A single worker polls it and serves
  everyone from Redis, so user growth does not raise this cost.
- DeepSeek V4.1 Flash API: the variable cost. Event-driven triggers and the pre-score keep it low.
- A VPS for the MVP: Ubuntu 22.04 or 24.04 LTS, at least 4 vCPU and 8 GB RAM, because DuckDB
  and Redis keep data in memory.
