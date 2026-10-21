# Open-Source Research

The research report covers 42 open-source projects. This page lists the ones FlowRadar reuses,
mapped to the three services. Reuse the code or the pattern. Do not rebuild.

## Mapping to the three services

| Service | Project | What FlowRadar takes | Pattern learned |
|---|---|---|---|
| Frontend | `awahab07/lightweight-orderflow-charts` | Footprint, TPO, POC, delta summary, CVD sub-chart on TradingView Lightweight Charts v5 | `OrderFlowBar` contract and mintick bucketing; dual-surface number formatting |
| Frontend | `tapedelta/kline-orderbook-chart` | Liquidation heatmap overlay | Separate drawing from data |
| Frontend | `Tucsky/aggr` | Web Worker that batches trades | Timeout-based aggregation: group ticks of the same millisecond before drawing |
| Gateway | `edgedepthhq/edgedepth-gateway` | Go collector for `aggTrade` and `depth`, Protobuf and Zstd packing | Pluggable venue interface and envelope pattern |
| Gateway | `apache/superset` websocket | Redis Pub/Sub fan-out | Message envelope and connection draining |
| Gateway | `Tucsky/aggr-server` | RAM buffer joined to a database | Hot buffer plus cold store merge |
| Python | `jose-donato/cryexc-backend` | DuckDB in-memory CVD and footprint | In-memory columnar computation |
| Python | `wickra-lib/wickra` | Two-pointer POC, VAH, VAL | Streaming-first indicators |
| Python | `ymys/Bot-Auto-Screening-Bybit-trading` | CVD divergence score, RVOL fakeout filter, trap filter | Deterministic pre-scoring |
| Python | `qrak/LLM_trader` | ChromaDB RAG, claim validation | Memory plus cross-checking the model's claims |
| Python | `lagarcess/crypto-signals`, `anvilfilbert/Auto-Crypto-Tradingjournal` | Lifecycle, multi-TP ladder, entry-drift guard | Managed trade lifecycle |
| Python | `mahmoud20138/OrderFlow-Analysis-Pro` | Rules for absorption, liquidity sweeps and delta divergence | Rule engine as pre-processor for the AI |
| Native app | `flowsurface-rs/flowsurface` | TPO math and DuckDB storage schema (a Rust desktop app, not a web app) | Value area calculation, Arrow IPC streams |

## Findings by area

**Presentation.** The charting layer should be a render-only library fed by a normalised data
contract. `lightweight-orderflow-charts` does exactly this, and it extends the TradingView chart
FlowRadar already uses.

**Transport.** A Go gateway with batching and a binary format handles tick volume that Node.js
and Python handle poorly. Redis Pub/Sub fans data out to many clients.

**Storage.** DuckDB in memory for the last day of ticks, TimescaleDB for history. A history request
must merge both, or the last candles go missing.

**Math.** Value area by the two-pointer method, and CVD divergence by comparing price swing lows
and highs with CVD swing lows and highs.

**AI.** Never let the model do the math. Score first, retrieve similar history, call the model,
lock its output with a schema, then recheck with code.

**Execution (deferred).** The report also covers bracket orders, risk-based position sizing
(`qty = risk_per_trade / |entry - stop|`), entry-drift guards, a dry-run switch, and an MCP server
for Bybit with strict Pydantic schemas. FlowRadar does not place orders (D1). Only the lifecycle,
the multi-TP ladder and the drift idea carry over, as signal tracking.

## Source

The full report is `Open Source Trading Architecture Research.docx` in the project root. Its
citations point to each project's GitHub page.
