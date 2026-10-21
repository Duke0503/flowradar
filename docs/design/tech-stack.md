# Tech Stack

## Frontend (`orderflow-frontend`)

| Concern | Choice | Reason |
|---|---|---|
| Framework | Next.js (App Router) or Vite with React | Vite suits a pure dashboard. Next.js suits a landing page that needs SEO |
| Language and styles | TypeScript, Tailwind CSS, shadcn/ui | Typed code, fast styling, accessible dropdowns, modals and toasts |
| Server data | TanStack Query | Caches REST history and the radar list. WebSocket events update the cache with `queryClient.setQueryData()` |
| Radar table | TanStack Table | Sorting and filtering thousands of signals by R:R or win rate |
| UI state | Zustand | Small. Holds the selected timeframe and which radar mode is on |
| Chart core | TradingView `lightweight-charts` v5 | Draws the K-line |
| Order flow drawing | `lightweight-orderflow-charts` | Footprint, TPO or market profile, CVD, delta summary |
| Heatmap | `kline-orderbook-chart` | Liquidation heatmap over the chart, if needed |
| Concurrency | HTML5 Web Workers | The worker receives and unpacks WebSocket data away from the main thread |

## Gateway (`orderflow-gateway-go`)

| Concern | Choice | Reason |
|---|---|---|
| Language | Go | Goroutines and channels handle tens of thousands of trades per second with no garbage-collection pauses like Node.js or Python |
| HTTP framework | Fiber or Gin | Fiber is very fast and reads like Express.js |
| WebSocket | `gorilla/websocket` | The standard Go library |
| Market data logic | Pattern from `edgedepth-gateway` | Listens to `@aggTrade` and `@depth`, batches 100 to 250 ms, packs Protobuf |
| Wire format | Protobuf with Zstd | Smaller and faster than JSON. About 70% less bandwidth |
| Client fan-out | Redis Pub/Sub | One subscription feeds thousands of browser tabs |

## AI engine (`orderflow-ai-python`)

| Concern | Choice | Reason |
|---|---|---|
| Framework | FastAPI | Async with `asyncio`, Pydantic built in |
| Quant | DuckDB in memory, NumPy, Pandas | Vectorised footprint, CVD and value area. Pattern from `cryexc-backend` |
| AI | DeepSeek V4.1 Flash with function calling or structured output | Cheap and quick |
| Output lock | Pydantic | A wrong format or wrong math is rejected |
| Long-term memory | ChromaDB with the `bge-base-en-v1.5` embedding model | Retrieves similar past setups and lessons |

## Storage

| Store | Role |
|---|---|
| Redis or Valkey | Hot cache of current price and live setups. Pub/Sub bus between Go and Python |
| PostgreSQL with TimescaleDB | History of candles (1m, 15m, 1H and larger), funding, open interest, and all setups for win rate and ROI boards |
| DuckDB | In-memory, inside Python. Not a system of record |
| ChromaDB | Vector memory of past setups |

## Why Go and Python split the work

| Layer | Language | Reason |
|---|---|---|
| Tick collector | Go | Concurrency and steady latency under heavy network load. `edgedepth-gateway` is a working Go example |
| Client gateway | Go | Reads from Redis and pushes to browsers with very low latency |
| Quant maths | Python | NumPy, Pandas and DuckDB do value area and divergence in a few lines. The same in Go is long and easy to get wrong |
| AI calls and RAG | Python | Pydantic and the vector database ecosystem. Go has little here |

The three services run separately. Go takes data in. Python computes and calls the AI. Go sends
the result out.

## Infrastructure

Linux VPS (Ubuntu 22.04 or 24.04 LTS), at least 4 vCPU and 8 GB RAM. Go and Python run in Docker.
The frontend deploys statically to Vercel, Netlify or a CDN. External services: CoinGlass Standard
API, Binance and Bybit WebSockets, DeepSeek API.
