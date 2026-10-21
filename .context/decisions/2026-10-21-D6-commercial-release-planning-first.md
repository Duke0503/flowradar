# D6 (FlowRadar is planned for commercial release; planning comes first)

- Status: accepted
- Date: 2026-10-21
- Approved by: the project owner, in the domain preparation session

## Context

The project owner intends to sell FlowRadar later, in Vietnam and internationally. Reused open-source
code, third-party data and signal regulation can each block a commercial release.

## Decision

FlowRadar is planned for commercial release in Vietnam and internationally. The team concentrates on
planning before implementation, and clears the licensing, data and regulation questions first.

## Rationale

These questions can change the design. If CoinGlass refuses public display, the data source and cost
model change. If a library is GPL-3.0, the code cannot be copied into a closed product.

## Consequences

- Code is learned from by pattern wherever the source is GPL-3.0, unlicensed or proprietary.
- Open questions OQ8 to OQ13 track the data, legal and licensing work.
- The checklist in `docs/commercialization.md` must be cleared before the first public release.
- Any feature that adds personal advice, copy trading or order routing reopens the regulation question.

## Links

- docs/licensing.md
- docs/commercialization.md
- docs/domain-model.json (decision `d6-commercial-release-planning-first`)
