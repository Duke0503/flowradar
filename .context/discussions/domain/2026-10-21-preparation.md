# Domain preparation session, 2026-10-21

## Participants

The project owner and the facilitator. Mary and Winston have not yet held a round.

## Loaded sources

- `Khởi động dự án Order Flow.pdf`: the design conversation.
- `Open Source Trading Architecture Research.docx`: the open-source research report.
- `bmad-domain-driven`, copied from another project and adapted (see below).

## Topic

Prepare the project for a first `bmad-domain-driven` session: decide the working language, answer
the open questions the sources left, and turn the two source files into English docs.

## What was set up

- The skill lives in `.claude/skills/bmad-domain-driven/`. Its rules, glossary and renderer were
  translated to English, and the glossary now holds FlowRadar's business language.
- `docs/` holds an index plus five documents: product vision, architecture, AI accuracy pipeline,
  open-source research and known issues.
- `docs/domain-model.json` holds decisions D1 to D6 and open questions OQ1 to OQ13. It has no
  contexts yet.

## Decisions approved by the project owner

- D1 (FlowRadar publishes signals only and never places orders)
- D2 (win rate counts only setups that became active; invalidated and expired setups are tracked separately)
- D3 (the Python Quant Engine owns value area computation and the TradeSetup lifecycle)
- D4 (the MVP is public and read-only, with no login or billing)
- D5 (rejected AI setups are stored with a rejection reason and never shown to users)
- D6 (FlowRadar is planned for commercial release in Vietnam and internationally; planning comes before implementation)

The project is written entirely in English. This is a working rule, recorded here and in `CLAUDE.md`.

## Open questions raised while reading the sources

OQ1 (the minimum R:R per timeframe), OQ2 (tightening targets versus the R:R gate), OQ3 (how Python
publishes computed value areas), OQ4 (the Redis channel naming rule), OQ5 (the CVD definition in the
draft code), OQ6 (the 2% drift limit), OQ7 (the expiry time of a pending setup). After the first commit, the question of commercial release added OQ8 (CoinGlass public display), OQ9 (exchange feed terms), OQ10 (signal regulation), OQ11 (GPL and unlicensed code), OQ12 (DeepSeek terms) and OQ13 (the liquidation heatmap source).

## Proposed starting model for the first round (not yet agreed)

- Core: Opportunity Radar, Quant Analysis.
- Supporting: Market Data, Delivery, Signal Memory.
- Generic: authentication and billing (deferred by D4), charting library, infrastructure.
- Context map: Market Data feeds Quant Analysis, which feeds Opportunity Radar. Opportunity Radar
  publishes setup events that Delivery forwards. Signal Memory listens to setup outcomes and feeds
  retrieval back to Opportunity Radar. An anti-corruption layer sits where CoinGlass and each
  exchange enter Market Data.

These are `proposed`. Mary and Winston test them in the first round.
