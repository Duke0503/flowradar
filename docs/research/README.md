# Research Reports

This folder holds the open-source research behind FlowRadar. The research studies 42 open-source projects
and extracts the patterns worth reusing. The scope covers five areas: charting, data pipeline,
quant math, AI orchestration, and trade execution with risk control.

The summary of what FlowRadar reuses is in [../open-source-research.md](../open-source-research.md).
These pages hold the detail behind that summary. Numbers in brackets, such as [12], point to
entries in [sources.md](sources.md).

## Reading order

1. [presentation-layer.md](presentation-layer.md): how the frontend draws footprint, heatmap and profile data.
2. [data-pipeline.md](data-pipeline.md): how ticks are collected, stored and fanned out.
3. [quant-engine.md](quant-engine.md): value area, CVD divergence and pre-scoring math.
4. [ai-radar.md](ai-radar.md): scoring, multimodal input, RAG memory and schema enforcement.
5. [recommendations.md](recommendations.md): the integration summary by layer.
6. [execution-and-risk.md](execution-and-risk.md): deferred reference material. Read last, and only if order execution returns to scope.
7. [sources.md](sources.md): the numbered citation list.

## Files

| File | Content | MVP status |
|---|---|---|
| [presentation-layer.md](presentation-layer.md) | Charting libraries, `OrderFlowBar` contract, project comparison table | In scope |
| [data-pipeline.md](data-pipeline.md) | Gateway, transport protocols, Redis Pub/Sub, time-series storage, replay | In scope |
| [quant-engine.md](quant-engine.md) | Value area algorithm, CVD divergence, machine learning for trading, indicator libraries | In scope |
| [ai-radar.md](ai-radar.md) | Opportunity scoring, DeepSeek integration, agentic RAG, Pydantic schemas, multi-agent projects | In scope |
| [execution-and-risk.md](execution-and-risk.md) | Bracket orders, sizing, drift guard, Multi-TP ladder, MCP exchange server | Deferred (decision D1) |
| [recommendations.md](recommendations.md) | Integration advice by layer and the adoption table | In scope |
| [sources.md](sources.md) | Numbered citations | Reference |

## Project context

- Decision D1: FlowRadar publishes signals only. The execution layer is deferred.
- Decision D3: the Python Quant Engine owns value area computation and the TradeSetup lifecycle.
- The research proposes a stack tendency by layer. Frontend uses TypeScript with React or Vue.
  The high-frequency transport layer moves to Rust or Go. Quant, machine learning and AI stay in
  Python with Pydantic, a vector database and LLM APIs such as DeepSeek or OpenAI.

## Note on source differences

- The research names InfluxDB 1.8 for `aggr-server`. FlowRadar uses TimescaleDB for history. See [data-pipeline.md](data-pipeline.md).
- The research suggests Vue or React for the frontend. FlowRadar uses Next.js (React).
- The research lists Tealstreet [3] only in its citations and does not discuss it.
- Citations 20, 25, 26, 33, 34, 36 and 38 are listed in the sources but not attached to any sentence in the report.
