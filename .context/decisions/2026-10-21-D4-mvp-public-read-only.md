# D4 (MVP is public and read-only)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

The research mentions paid access and many users. Auth and billing add scope that does not differentiate the product.

## Decision

The MVP is public and read-only, with no login and no billing.

## Rationale

Auth and billing are a generic subdomain. The fixed CoinGlass cost does not grow with users, and DeepSeek cost is controlled by event-driven calls.

## Consequences

Access control, when added, is a policy on the Delivery context's read model. It does not touch the TradeSetup aggregate.

## Links

- docs/domain-model.json (decision `d4-mvp-public-read-only`)
- docs/known-issues.md
