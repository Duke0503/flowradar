# D3 (Python owns value area and the setup lifecycle)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

Go collects ticks and Python computes TPO. The chart also draws POC, VAH and VAL. Setup status must change from live price. Without one owner, two copies of each number can differ.

## Decision

The Python Quant Engine is the only place that computes POC, VAH, VAL and CVD. The Opportunity Radar in Python is the only writer of TradeSetup status changes. Go does no analysis. Go publishes a throttled price channel (about 4 per second) and bar-close events, and Python compares price to each setup's levels.

## Rationale

One owner means the chart and the AI always see the same numbers. Setups follow 15m and 4H bars, so a status check does not need 50 ms latency.

## Consequences

Python must publish computed bars and value areas to Redis for Go to forward. How it does so is OQ3. If latency later matters, Go may emit a level-crossed event, but Python stays the decider.

## Links

- docs/domain-model.json (decision `d3-python-owns-quant-and-lifecycle`)
- docs/known-issues.md
