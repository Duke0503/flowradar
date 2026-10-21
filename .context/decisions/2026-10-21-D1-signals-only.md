# D1 (FlowRadar publishes signals only)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

The research report describes an execution and risk layer: bracket orders, position sizing and exchange MCP servers. The product could stop at signals or go on to place orders.

## Decision

FlowRadar publishes trade signals only. It never places orders and never holds exchange API keys.

## Rationale

The MVP scope stays small. The project avoids liability for users' orders and money. The edge is the quality of the signal, not execution.

## Consequences

There is no Execution context in the model. It is recorded as deferred so it can be added later. The ACTIVE state of a TradeSetup means price entered the entry range, not that a user took the trade.

## Links

- docs/domain-model.json (decision `d1-signals-only`)
- docs/known-issues.md
