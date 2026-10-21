# Recommendations

The report concludes that building an order flow platform with an AI radar from scratch carries high
cost and maintenance risk. The open-source projects here can be combined. To keep latency very low
and still allow complex reasoning, the report recommends a distributed microservice design.

## Frontend layer

- Use Vue.js or React with `lightweight-orderflow-charts` to render DOM charts, Footprint profiles and CVD [1].
- The `OrderFlowBar` data contract separates data from the drawing surface. This raises client performance.
- Use `kline-orderbook-chart` for the orderbook heatmap [1].

FlowRadar choice: Next.js (React) with TradingView Lightweight Charts v5.

## Streaming backbone

- Do not send large volumes as JSON over plain WebSockets.
- Use a Go gateway like `edgedepth-gateway` [12] and Arrow IPC streams as in `flowsurface` [9].
  These reduce bandwidth pressure and carry very small tick data smoothly.
- Use in-memory DuckDB as in `cryexc-backend`. It resamples and computes the Value Area and CVD as data arrives [19].
- Use a multi-stream Redis Pub/Sub cluster to distribute data to end users [14].

## AI orchestration

- Reserve Python for the machine learning and quant layer.
- Follow the `LLM_trader` flow [29] and multi-model routing to wrap DeepSeek Flash in strict controls.
- ChromaDB gives long-term memory. The radar adapts and learns after each trade [29].

## Risk controls

Deferred under D1. Kept as reference. See [execution-and-risk.md](execution-and-risk.md).

- Close the exchange API touchpoint with Pydantic wrappers and an MCP server [32].
- Use hard-coded multi-tier bracket orders (Multi-TP), sizing from stop distance, and slippage
  tolerance (entry-drift guards). These neutralise financial errors from AI hallucination [31].

Note: the report cites [31] and [32] for this paragraph. Check the cross references in
[ai-radar.md](ai-radar.md).

## Adoption table

| Layer | Projects to adopt | Why |
|---|---|---|
| Frontend | `lightweight-orderflow-charts`, `kline-orderbook-chart`, `aggr` (batching idea) | Render-only chart on a normalised `OrderFlowBar` contract. Heatmap of the orderbook and liquidations. Tick batching keeps the browser responsive |
| Streaming backbone | `edgedepth-gateway`, `flowsurface-rs` (Arrow IPC), `cryexc-backend`, `superset-websocket`, `aggr-server`, `tardis-node` | Binary Go transport. Fast history loading. In-memory CVD and value area math. Redis fan-out with draining. Hot buffer plus cold store merge. Replay with the live format |
| Quant engine | `wickra`, `OrderFlow-Analysis-Pro`, `Bot-Auto-Screening-Bybit-trading` (`bybit_bot_v8` logic) | Streaming value area. CVD divergence and deterministic pre-scoring |
| AI orchestration | `LLM_trader`, Pydantic schemas, ChromaDB, `TradingAgents`, `swarm-trader` | Multimodal flow with claim validation. Schema-locked output. Long-term memory. Provider-agnostic routing |
| Risk controls (deferred, D1) | `crypto-signals`, `Auto-Crypto-Tradingjournal`, `schwagent`, `bybit-py-mcp`, `CryptoSignalApp` | Bracket orders, drift guard, Multi-TP, dry-run, MCP schemas. Only the lifecycle ideas carry over as signal tracking |
