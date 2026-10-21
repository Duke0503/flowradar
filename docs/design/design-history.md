# Design History

The design came from one conversation. This page lists the turns in order, with the result of each.
Decisions D1 to D5 were made later and are not part of the conversation.

## 1. First request: review open source, then give the full flow

**Ask.** Review the best open-source projects of this type, suggest upgrades, and give the full flow
of the site with its issues and solutions.

**Result.**
- No single project has both order flow charting and an AI radar. Four projects cover about 80% of
  the data and chart work: `Tucsky/aggr` with `aggr-server`, `awahab07/lightweight-orderflow-charts`,
  `flowsurface-rs/flowsurface`, and `OrderFlow-Analysis-Pro`. See
  [../open-source-research.md](../open-source-research.md).
- Three upgrades: draw AI setups on the chart as boxes, call the AI on events instead of a timer,
  and track every setup live with automatic invalidation.
- A four-layer flow: data ingestion, storage and quant pre-processing, the AI radar engine, and
  frontend delivery.
- Five system issues with solutions: CoinGlass rate limit, TPO and footprint lag, invented price
  levels, conflicting intraday and HTF signals, and zombie signals. See [../known-issues.md](../known-issues.md).

## 2. What does "real time" mean here

**Result.** Each part has its own speed. Exchange trades move in 50 to 250 ms. CoinGlass data moves
every 5 to 30 seconds. One AI analysis takes 1.5 to 3 seconds and runs on events. All timestamps are
UTC Unix milliseconds, and only the browser converts to local time.

## 3. Research all open source and learn from it

**Result.** A research plan was approved and a report on 42 projects was produced. It is
converted in [../research/README.md](../research/README.md).

## 4. Architecture and tech stack

**Result.** A first stack: Next.js with TypeScript, Tailwind, Lightweight Charts and the orderflow
plugin, Web Workers, FastAPI with a Node.js worker, Redis, DuckDB with TimescaleDB, and DeepSeek
V4.1 Flash with Pydantic. The design is event-driven with four layers.

## 5. Go or Python

**Question.** Should the data layer use Go?

**Result.** Yes. Go takes the network and data layers (tick collector and client WebSocket
gateway) and replaces Node.js. Python keeps the quant and AI layers because NumPy, Pandas,
DuckDB, Pydantic and vector databases live there. Go and Python meet through Redis. See
[tech-stack.md](tech-stack.md).

## 6. Complete stack, with TanStack

**Question.** Can the frontend use TanStack?

**Result.** Yes. TanStack Query caches API data and takes WebSocket updates through
`setQueryData`. TanStack Table shows the radar. Zustand holds UI state. See [tech-stack.md](tech-stack.md).

## 7. How many repositories

**Question.** Should the frontend, the backend and the AI be separate projects?

**Result.** Three repositories: frontend, Go gateway, Python AI engine. See [repositories.md](repositories.md).

## 8. How do the services talk

**Result.** Four paths. REST for history, WebSocket for live data, Redis Pub/Sub for the data
flow between Go and Python, gRPC for direct commands. See [communication.md](communication.md).

## 9. Lock the architecture

**Result.** A final blueprint with the three repositories, a storage layer (Redis hot, TimescaleDB
cold), and the communication map. This is the architecture in [../architecture.md](../architecture.md).

## 10. Research again, map open source to each repository

**Result.** A second report maps each repository to the projects and patterns it should inherit.
It also defines the five-layer accuracy pipeline in [../ai-accuracy-pipeline.md](../ai-accuracy-pipeline.md).

## 11. First code

**Ask.** Write complete Python for layers 1, 2 and 3.

**Result.** A draft class `OrderFlowQuantEngine`. It is kept, with its defects, in
[quant-engine-draft.md](quant-engine-draft.md).

## Changes since the conversation

| Topic | In the conversation | Now |
|---|---|---|
| Order placement | A research layer for execution | Out of scope (D1) |
| Win rate | Not defined | Active setups only (D2) |
| Value area owner | Go and Python both touch it | Python only (D3) |
| Users | Public site with many users | Public and read-only for the MVP (D4) |
| Rejected AI setups | Dropped | Stored with a reason (D5) |
| Setup states | PENDING_TRIGGER, ACTIVE, HIT_TP, HIT_SL, INVALIDATED | Adds EXPIRED (D2) |
