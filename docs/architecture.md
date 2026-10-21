# Architecture

## Overview

FlowRadar is three independent services connected by Redis and gRPC. The split follows what each
part is good at, and it lets the AI prompts change without touching the live chart.

| Repository | Stack | Role |
|---|---|---|
| `orderflow-frontend` | Next.js or Vite React, TypeScript, Tailwind, shadcn/ui, TanStack Query and Table, Zustand, TradingView Lightweight Charts v5 with `lightweight-orderflow-charts`, Web Workers | Draws charts and the radar. Does no heavy computation |
| `orderflow-gateway-go` | Go, Fiber or Gin, Gorilla WebSocket, go-redis, Protobuf with Zstd | Takes ticks from Binance and Bybit, polls CoinGlass, batches, and fans out to browsers |
| `orderflow-ai-python` | Python, FastAPI, DuckDB, NumPy and Pandas, Pydantic, ChromaDB, DeepSeek V4.1 Flash | Computes order flow metrics, scores candidates, calls the AI, validates, owns setups |

Storage and transport:

- **Redis (or Valkey):** hot cache of current price, current bar and live setups, and the Pub/Sub
  bus between Go and Python.
- **PostgreSQL with TimescaleDB:** history of candles, funding and open interest, and the full
  history of setups and outcomes.
- **DuckDB:** in-memory, inside the Python service, for fast footprint, CVD and value area math.
- **ChromaDB:** vector memory of past setups for retrieval.

## How the services talk

| Path | Protocol | Why |
|---|---|---|
| Browser to Go, history | REST over HTTP/2 | One large pull at page load, such as one month of candles and the last 50 setups. TanStack Query caches it |
| Go to browser, live | WebSocket (WSS) | Go pushes ticks, footprint columns and setup events without being asked |
| Go to Python, market data | Redis Pub/Sub or Streams | Go handles about 10,000 trades per second. gRPC at that rate would overwhelm Python |
| Python to Go, setups | Redis Pub/Sub | Python publishes to `channel:ai_signals`, and Go forwards to browsers |
| Go to Python, on demand | gRPC with Protobuf | For direct commands that need an answer now, such as "rescan ETH now" |

Every Redis message uses one envelope: `topic`, `scope`, `payload`. Topics follow the pattern
`<domain>:<symbol>:<kind>`, for example `orderflow:btcusdt:1m` and `radar:signal:new`. The source
documents use inconsistent channel names. See [known-issues.md](known-issues.md).

## Data flow

```
CoinGlass API  --(10-30 s: OI, funding, liquidations)--+
                                                        v
Binance/Bybit WS --(real time: trades)--> Go gateway (batch 100-250 ms, Protobuf+Zstd)
                                                        |
                                                        v
                                                      Redis
                              +-------------------------+--------------------------+
                              v                                                    v
                    Fast path: chart data                              Slow path: AI Radar (Python)
                    Go WebSocket fan-out                      DuckDB -> quant metrics -> pre-score
                              |                                    -> RAG -> State Vector -> DeepSeek
                              v                                    -> guardrail -> setup lifecycle
                    Browser Web Worker                                                |
                    -> Lightweight Charts                                             v
                              ^                                     Redis + TimescaleDB, then Go
                              +---------------- setup overlay -------------------------+
```

1. **Ingest.** Go receives `aggTrade` and `depth` streams, groups trades into 100 to 250 ms
   batches, and writes to Redis. A separate Go poller reads CoinGlass every 10 to 30 seconds.
2. **Refine.** Python loads the last 24 hours of trades into DuckDB, builds OrderFlowBars, and
   computes POC, VAH, VAL, CVD, RVOL and OI change.
3. **Filter.** A deterministic pre-score decides whether the AI is called at all.
4. **Reason and validate.** The AI returns a setup, and the guardrail accepts or rejects it.
5. **Deliver.** Accepted setups go to TimescaleDB and Redis, and Go pushes them to browsers.

## Latency budget

| Stage | Target |
|---|---|
| Exchange trade to browser | 50 to 250 ms. The browser Web Worker batches 250 ms and renders four times a second |
| CoinGlass metrics (OI, funding, liquidations) | 5 to 30 s |
| One AI analysis | 1.5 to 3 s, on a bar close or an anomaly, not on every tick |
| Setup status check (trigger, TP, SL) | Under 50 ms in the source design. See D3 and [known-issues.md](known-issues.md) |

All timestamps are stored and processed as UTC Unix milliseconds. Only the browser converts to
local time. This keeps 4H and daily candle closes consistent worldwide.

## Frontend rules

- The Web Worker receives the binary stream, groups ticks for about 200 ms, and calls
  `series.update()` on the chart directly, bypassing React re-renders.
- New setups and status changes arrive as JSON events and go straight into the TanStack Query
  cache with `queryClient.setQueryData(['ai-radar-signals'], updater)`. TanStack Table then
  updates the right row.
- Zustand holds local UI state only: selected timeframe, radar filters.
- `clusterOrderFlowBarsByMintick` fills sparse price levels so footprint cells line up with candles.
- Price axis and footprint cells format numbers separately: `64,250.50` on the axis, `125.4k`
  inside a cell.

## Gateway rules

- Venues are pluggable. Each venue adapter turns its feed into one standard trade shape.
- A history request merges TimescaleDB with the unflushed buffer in Redis, so the last few
  seconds of candles are never missing.
- The `/ready` endpoint reports failure when Redis is down, so the load balancer stops sending
  users to a gateway that would show stale data.
- Protobuf with Zstd cuts bandwidth from the VPS to the browser by about 70% compared with JSON.

## Infrastructure for the MVP

Linux VPS, Ubuntu 22.04 or 24.04 LTS, at least 4 vCPU and 8 GB RAM. Each service runs in its
own Docker container. The frontend deploys statically to Vercel, Netlify or a CDN.
