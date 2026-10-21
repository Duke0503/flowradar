# Repositories

FlowRadar is three repositories. They differ in language, deployment and reason to change, so they
stay apart. A new prompt can ship without a restart of the live chart.

## 1. `orderflow-frontend`

**Role.** Show the chart and the radar. No heavy computation. Target 60 FPS.

**Deploy.** Static, on Vercel, Netlify or a CDN.

```
orderflow-frontend/
├── src/
│   ├── components/
│   │   ├── charts/        Canvas components: K-line, Footprint, TPO
│   │   ├── radar/         AI Opportunity UI, TanStack tables
│   │   └── ui/            Buttons, modals, layout (shadcn/ui)
│   ├── hooks/             useWebSocket, TanStack Query hooks
│   ├── workers/           Web Workers that unpack WebSocket data off the main thread
│   ├── store/             Zustand local state
│   └── services/          API and WebSocket clients
├── package.json
└── tailwind.config.js
```

## 2. `orderflow-gateway-go`

**Role.** The network workhorse. It takes ticks from the exchanges, batches and packs them, and
pushes to thousands of browsers.

**Deploy.** Docker on a VPS. It scales out by running more copies.

```
orderflow-gateway-go/
├── cmd/
│   └── server/            Entry point (main.go)
├── internal/
│   ├── exchange/          Binance and Bybit WebSocket logic (@aggTrade, @depth)
│   ├── pubsub/            Redis publish and subscribe
│   ├── client_ws/         Thousands of browser WebSocket connections
│   └── models/            Protobuf and struct definitions
├── pkg/                   Shared tools: logger, config
├── go.mod
└── Dockerfile
```

The CoinGlass poller also lives in this repository. It calls the API every 10 to 30 seconds for
open interest, funding rate and liquidation levels.

## 3. `orderflow-ai-python`

**Role.** The brain. It does not care how users reach the site. It reads market data from Redis,
computes, calls the AI, validates and owns the setup lifecycle (D3).

**Deploy.** Docker on a VPS. It needs a strong CPU and enough RAM, because DuckDB runs in memory.

```
orderflow-ai-python/
├── main.py                FastAPI entry (triggers and cron jobs)
├── core/
│   ├── quant_engine/      DuckDB: TPO (POC, VAH, VAL) and CVD divergence
│   ├── ai_radar/          DeepSeek V4.1 Flash client
│   └── prompts/           System prompts that shape the AI
├── schemas/               Pydantic models: entry, stop loss, take profit
├── database/
│   ├── redis_client.py    Listen for new data, store AI signals
│   ├── timescaledb.py     Store setup history and 15m and 1H candles
│   └── vectordb/          ChromaDB memory of past trades
├── requirements.txt
└── Dockerfile
```

## Shared contracts

The three repositories share message shapes and Protobuf files. The conversation does not say where
they live. Recommendation: keep them in one place so the three stay in step, and settle this when
the domain model defines the shared kernel. See [data-contracts.md](data-contracts.md).
