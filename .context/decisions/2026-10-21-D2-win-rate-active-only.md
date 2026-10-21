# D2 (Win rate counts active setups only)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

A setup can end invalidated or expired before price ever enters its entry range. Counting those as losses or wins would distort the win rate.

## Decision

Win rate counts only setups that became active and then ended at a take-profit. Invalidated and expired setups are tracked separately as the invalidation rate. Results are stored as R-multiples. A new EXPIRED state covers setups that time out before triggering.

## Rationale

A setup that never entered is not yet a trade, so it is neither a win nor a loss. Multi-TP setups make a binary result disputed, so an R-multiple is the fair measure.

## Consequences

The lifecycle gains EXPIRED. Signal Memory stores the R-multiple and the final state. The expiry time per timeframe is open (OQ7).

## Links

- docs/domain-model.json (decision `d2-win-rate-active-only`)
- docs/known-issues.md
