# Communication Between Parts

No single protocol fits every path. The choice follows the nature of the data: asynchronous bulk
flow, or a synchronous request that needs an answer now.

## Map

| # | Path | Protocol | Use |
|---|---|---|---|
| 1 | Browser asks Go for history | REST over HTTP/2 | Once at page load or on a timeframe change |
| 2 | Go pushes live data to the browser | WebSocket (WSS) | Ticks, footprint columns, new and updated setups |
| 3 | Go to Python, market data | Redis Pub/Sub or Streams | The main flow. Asynchronous and decoupled |
| 4 | Python to Go, setups | Redis Pub/Sub | Python publishes to a signals channel and Go forwards |
| 5 | Go calls Python directly | gRPC with Protobuf | A command that needs an immediate result |

## 1. Browser and Go: REST and WebSocket

- **REST** loads a large block once. Examples: one month of candles, or the 50 most recent setups.
  TanStack Query stores it in its cache.
- **WebSocket** opens right after. From then on Go pushes new ticks, footprint columns and settled
  setups without being asked.
- A history request merges TimescaleDB with the unflushed buffer in Redis, so the last candles are
  never missing.

## 2. Go and Python: Redis Pub/Sub (main flow)

- Go receives about 10,000 trades per second. Calling Python over gRPC 10,000 times per second would
  freeze the Python server, because Python handles concurrency far less well than Go.
- **How it decouples.**
  1. Go batches trades and publishes them to a Redis channel. Go does not care whether Python is free.
  2. Python subscribes, and loads the data into DuckDB. Every 15 minutes, or on an event, it
     computes the profile, calls the AI, and publishes the setup to a second channel.
  3. Go listens on that second channel and forwards each new setup to browsers over WebSocket.

## 3. Go and Python: gRPC (commands)

- Use gRPC when Go must tell Python to do something now and wait for the result. Example: a user
  presses "scan ETH now".
- gRPC sends Protobuf over HTTP/2. The source claims it is 7 to 10 times faster than internal REST.
  Both Go and Python support Protobuf well.

## Message envelope

Every message through Redis uses one wrapper with `topic`, `scope` and `payload`. See
[data-contracts.md](data-contracts.md). A gateway whose Redis connection fails reports not ready,
so users never see frozen data without knowing.

## Rule from decision D3

Python is the only writer of setup status. Go sends a throttled price channel (about four updates
per second) and bar-close events. Python compares price to each setup's levels and publishes every
status change.

## Open points

- Channel names are inconsistent in the sources. See OQ4 in [../known-issues.md](../known-issues.md).
- How Python publishes computed bars and value areas is OQ3.
