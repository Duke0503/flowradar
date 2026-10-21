# D5 (Rejected AI setups are stored)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

The guardrail rejects AI setups that fail schema, R:R, stop placement or claim checks. The source documents say only that they are rejected.

## Decision

A rejected setup is stored with its rejection reason. It is never shown to users.

## Rationale

The rejection history is the best data for tuning prompts and pre-score thresholds.

## Consequences

Signal Memory keeps rejected setups next to accepted ones, with a reason. Retrieval may learn from them.

## Links

- docs/domain-model.json (decision `d5-store-rejected-setups`)
- docs/known-issues.md
