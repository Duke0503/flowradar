# Data Pipeline

An order flow platform needs heavy transport and storage. In crypto and futures, tick data and
continuous orderbook updates can reach gigabytes per day for one pair. Rebuilding a time-series
database or a multi-channel WebSocket router is wasteful and risky. The projects below give a
reference design from network protocol to database.

## Gateway and transport protocol

### edgedepth-gateway

Repository: `edgedepthhq/edgedepth-gateway` (Go) [12].

- Collects public data from Binance USD-M Futures, such as `@aggTrade` and `@depth`.
- Converts it to Protobuf before sending over WebSocket to clients. Protobuf removes repeated text
  keys, so bandwidth and parsing latency drop compared with JSON.
- Zstd compression sniffing: it detects packets already compressed and keeps them compressed when
  the client supports it.
- Venues are pluggable behind an `Exchange` interface. A new venue such as Bybit, OKX or
  Hyperliquid needs one adapter package.
- A REST-polled Stats feed delivers mark price, funding rate, open interest and liquidation data.

### tardis-node

Repository: `tardis-dev/tardis-node` [13].

- Switches seamlessly between real-time streaming and historical replay.
- Replay uses the exchange native data format.
- Models then train on the same micro-data format they meet in production. This removes the
  sim-to-real gap between test and production.

## Redis Pub/Sub multichannel fan-out

Reference projects: `apache/superset` `superset-websocket` [15] and
`redis-developer/basic-redis-chat-app-demo-python` [14]. The sources also list
`permitio/fastapi_websocket_pubsub` [16] and `nijatsdev/marketfeed` [17] without discussing them.

Design [14]:

- A Redis (or Valkey) server is the central bus.
- A cluster of backends, for example Python FastAPI, publishes to dedicated Redis channels.
- WebSocket workers, usually Node.js for its async event loop, subscribe to Redis. They fan out
  messages to browsers or agents holding a WebSocket connection.

Message envelope in `superset-websocket` [15]:

| Field | Role |
|---|---|
| topic | Subject of the message |
| scope | Reach of the message |
| routes | Routing |
| payload | Data |

The envelope separates the nature of a message from the choice of recipients.

Example flow [15]: on a new tick, the backend sends JSON to a Redis channel such as
`marketfeed:tick:BTCUSDT` with scope `authenticated_global`. Every WebSocket worker connected to
Redis receives it and pushes it to the matching connections.

Connection draining on Kubernetes [15]:

- The Redis connection state is tied to the `/ready` endpoint, not `/health`.
- If a pod loses its Pub/Sub connection, Kubernetes stops routing new browser connections to it until it recovers.
- The radar therefore never serves stale data.

## Time-series storage

### aggr-server

Repository: `Tucsky/aggr-server` [18]. The report also cites the older SignificantTrades README [7].

- Cluster design with InfluxDB 1.8 in Docker.
- Collectors listen to many exchanges and write directly to InfluxDB.
- An API node serves clients. It reads InfluxDB and also connects to the collectors' buffers to
  get data "NOT YET stored" on disk. History and live data then join without a gap.

FlowRadar takes the pattern, not the database: a hot buffer joined to a cold store. FlowRadar
uses TimescaleDB for history and DuckDB in memory for recent ticks. A history request must merge
both.

### cryexc-backend

Repository: `jose-donato/cryexc-backend` [19].

- Uses DuckDB, an in-process analytical database that runs fully in memory.
- Stores all Binance Futures WebSocket streams in RAM: `@trade`, `@depth@100ms`, `@forceOrder`.
- Deletes trades older than 24 hours once per hour.
- Uses vectorised processing to resample millions of raw trades into Footprint volume (by time
  interval and price step) and to compute Cumulative Volume Delta (CVD). There is no disk I/O delay.

### flowsurface-rs Arrow IPC

Repository: `flowsurface-rs/flowsurface` [9].

- A self-hosted server (for example DuckDB with Rust) exports raw data as Apache Arrow columns
  over HTTP. The data holds time, price, quantity and buy/sell flag.
- The client loads it without CPU-heavy parsing (zero-copy deserialization).
- Reloading order flow data for large time frames becomes almost instant.

## Summary table

| Project | Role | Pattern to take |
|---|---|---|
| `edgedepth-gateway` | Go collector | Protobuf, Zstd sniffing, pluggable venues, REST Stats |
| `tardis-node` | Replay | Same format for live and replay |
| `superset-websocket` | Fan-out | Message envelope, connection draining through `/ready` |
| `aggr-server` | Storage | Buffer and database merge |
| `cryexc-backend` | In-memory compute | DuckDB with 24 hour retention |
| `flowsurface-rs` | History transfer | Arrow IPC over HTTP |
