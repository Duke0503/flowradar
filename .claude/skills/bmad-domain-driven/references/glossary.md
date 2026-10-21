# Glossary: FlowRadar business language and the DDD terms used in the model, the shared page and the Mary/Winston discussions

> Single source for names. The renderer (`BUSINESS_GLOSSARY` and `GLOSSARY` in
> `scripts/render_domain_model.py`) and everything an agent says follow these two tables. The
> project is written entirely in English. Use DDD terms as written here; do not invent synonyms.
> Business names (aggregate, event, what a rule is about) are plain English trading words.

## Business language (the ubiquitous language of FlowRadar)

> Words every page and every agent sentence uses with exactly one meaning. Write the way the
> team talks: short sentences, a subject, what happens. The single source is `BUSINESS_GLOSSARY`
> in `scripts/render_domain_model.py`; the table below mirrors it. When the domain changes,
> edit both.

| Term | Meaning |
|---|---|
| **Order flow** | The record of who bought and who sold at each price. FlowRadar reads it to see pressure, not just price |
| **Tick / aggTrade** | One executed trade from an exchange. Carries price, size and which side was the aggressor |
| **Aggressor (taker)** | The side that crossed the spread. A buy aggressor lifts the ask; a sell aggressor hits the bid |
| **Venue** | An exchange feed such as Binance or Bybit. Each venue has its own symbols and quirks, normalised on entry |
| **Symbol** | One tradable instrument on a venue, for example BTCUSDT |
| **OrderFlowBar** | One candle plus the volume traded at every price level inside it, split by buy and sell. The standard contract between data and chart |
| **Mintick bucket** | The smallest price step used to group levels so footprint cells line up with the candle |
| **Footprint** | The chart view of an OrderFlowBar: buy and sell volume at each price level |
| **Delta** | Buy aggressor volume minus sell aggressor volume over a period |
| **CVD (cumulative volume delta)** | The running sum of delta. Shows whether buyers or sellers are in control over time |
| **CVD divergence** | Price makes a new low while CVD makes a higher low, or the reverse. Signals that aggression is being absorbed |
| **Absorption** | Large passive orders soak up aggressive flow without price moving. Often marks a turn |
| **Liquidity sweep** | Price runs through a cluster of stops or liquidations, then reverses |
| **TPO / market profile** | Time spent at each price over a session, drawn as a profile |
| **Volume profile** | Volume traded at each price over a session |
| **POC (point of control)** | The price level with the most volume or time in the profile |
| **Value area (VAH / VAL)** | The price range holding 70% of the profile around the POC. VAH is its top, VAL its bottom |
| **Open interest (OI)** | The number of open futures contracts. A rising OI with price means new positions, not just covering |
| **Funding rate** | The periodic payment between longs and shorts on perpetual futures. A very positive rate means a crowded long |
| **Liquidation level / heatmap** | Prices where leveraged positions are forced to close. Acts as a magnet for price |
| **Long/short ratio** | The share of accounts or volume positioned long versus short |
| **RVOL (relative volume)** | Current volume divided by its normal level. A breakout with low RVOL is suspect |
| **HTF bias** | The direction the higher timeframes (4H, 1D) lean. Intraday setups are judged against it |
| **Intraday / HTF scan** | Intraday runs on 15m bars for scalp and day trades. HTF runs on 4H and 1D bars for swing trades |
| **State vector** | The compact JSON snapshot of the market (about 500 tokens) sent to the model: levels, flow signals, bias |
| **Anomaly trigger** | A market event that justifies calling the AI: a bar close, a large liquidation spike, or a CVD divergence at the POC |
| **Pre-score** | A deterministic score from hard rules. A candidate must pass it before the AI is called |
| **Trap filter** | A hard rule that discards a candidate that looks like exhaustion, such as a long while funding is crowded long |
| **Opportunity Radar** | The feed of AI-found trade setups, filterable by intraday or HTF |
| **TradeSetup** | One recommended trade: direction, trigger, invalidation, entry range, stop loss, take-profit targets, R:R and confidence. A recommendation only. FlowRadar never places the order |
| **Trigger condition** | What must happen before a setup becomes live, for example a 5m close above a level with positive delta |
| **Invalidation condition** | What kills the setup before it triggers, for example price holding below the VAL |
| **Entry range / stop loss / take-profit (TP1..TP3)** | The levels of a setup. Each must sit on a verified structural level, never a number the AI invented |
| **R:R (risk-to-reward)** | Reward distance divided by risk distance. A setup needs R:R of at least 2.0 |
| **R-multiple** | A finished setup's result measured in units of risk, for example +2.1R or −1R |
| **Anchor level** | A level computed by the Quant Engine (VAL, POC, VAH, liquidation cluster) offered to the AI as the only allowed prices |
| **Guardrail** | The deterministic check that accepts or rejects an AI setup: schema, R:R, stop placement, claim check |
| **Claim validation** | Cross-checking what the AI says against measured data, for example "strong trend" against ADX. A mismatch lowers confidence |
| **Rejected setup** | An AI setup the guardrail refused. Stored with a rejection reason; never shown to users |
| **Zombie signal** | A setup left on screen after price already passed its target or stop. The lifecycle exists to prevent it |
| **Invalidated / expired** | Invalidated: price hit the invalidation before entry. Expired: the setup timed out before triggering. Neither counts as a win or a loss |
| **Win rate** | Share of setups that became active and then ended at a take-profit. Invalidated and expired setups are tracked separately |
| **Invalidation rate** | Share of setups that ended invalidated or expired before entry |
| **Signal memory** | The stored history of setups and outcomes, plus post-mortem lessons, retrieved to inform later AI calls |
| **Post-mortem lesson** | A short note on why a stopped-out setup failed, saved for the AI to see next time |
| **Hot buffer / cold store** | Hot: the latest unflushed data in Redis. Cold: history in TimescaleDB. A history request merges both |
| **Envelope** | The standard wrapper on every message through Redis: topic, scope and payload |
| **Venue adapter** | The piece that turns one exchange's feed into FlowRadar's standard trade shape |
| **Workspace** | The chart area plus the radar feed the user sees. 70% chart, 30% radar |

## Strategic design

| Term | Short meaning | Usage note |
|---|---|---|
| **Domain** | The business area the system serves | "the Opportunity Radar domain" |
| **Subdomain** | A part of the domain, ranked by how much to invest | core / supporting / generic |
| **Core** | What makes FlowRadar different; invest deeply | keep the word "core" |
| **Supporting** | Needed, but every competitor needs it too; build just enough | keep the word "supporting" |
| **Generic** | Plumbing with no business edge; buy or keep thin | keep the word "generic" |
| **Bounded Context** | The boundary of one model and its language; the same word may mean something else in the next context | short form "context"; here a context is usually a service or package |
| **Ubiquitous Language** | The vocabulary shared by the business and the code inside one context | the "Ubiquitous language" section on the page |
| **Context Map** | The map of which context calls, listens to or references which | |
| **Open-host service** | A context opens a stable API for many callers | relationship pattern |
| **Published language** | A context publishes events or contracts others listen to | relationship pattern |
| **Customer–supplier** | Two sides negotiate the contract; the consumer is the customer | relationship pattern |
| **Conformist** | The consumer accepts the supplier's language and does not translate | relationship pattern |
| **Anti-corruption layer (ACL)** | A layer that translates an outside language into our own | also a building block |
| **Shared kernel** | A small part of the model that two sides own and change together | e.g. the Protobuf message definitions |
| **Partnership / Upstream–downstream / Separate ways** | Evolve together / one side leads / go separate ways | relationship pattern |

## Tactical design (building blocks)

| Term | Short meaning | Usage note |
|---|---|---|
| **Building block** | A named DDD piece in the model (aggregate, entity, value object, service…) | |
| **Aggregate** | A group of data that must be correct together; every change goes through one door | "the TradeSetup aggregate" |
| **Aggregate root** | The record at the head of an aggregate and its only entry point | |
| **Entity** | Something with its own identity that lives over time | |
| **Value object** | A value with no identity, compared by content (Price, Money) | |
| **Domain primitive** | A single constrained value type (Symbol, PriceLevel) | |
| **Invariant** | A condition that must never be false inside an aggregate | the "Invariants" section |
| **Rule** | A named business rule; classified by `kind` below | |
| **Policy** | A named decision that changes with context or time ("a counter-trend setup needs higher confidence") | |
| **Specification** | A reusable test ("a setup is publishable when…") | |
| **Authorization** | A rule about who may do what | rule kind |
| **Uniqueness** | A rule against duplicates | rule kind |
| **Workflow** | A rule about the order of steps | rule kind |
| **Lifecycle** | The states an aggregate passes through and the steps between them | "the TradeSetup lifecycle" |
| **State / Transition / Guard** | A state / a step between states / a condition that must hold before the step | |
| **Command** | A request to do something (`confirm`, `approve`); can be refused | the "Commands" section |
| **Use case** | What a user does with the domain, usually several commands | |
| **Event** (domain event) | Something that already happened, in past tense (`radar.setup.activated`); other contexts can react | the "Events" section |
| **Domain service** | A business operation that belongs to no single aggregate | |
| **Repository** | The door for reading and writing one aggregate | |
| **Factory** | The place that creates an aggregate when creation has many rules | |
| **Port** | An interface a context declares to call out or to be called | "the Market Data port" |
| **Adapter** | The implementation of a port for one concrete system (Binance feed, CoinGlass API, DeepSeek API) | |
| **Process manager** | A process that coordinates several aggregates over time (the job that expires stale setups) | |
| **Read model** | Read-only data built from another source for display or reports; holds no rules | |
| **EventStorming** | Reading a domain as sentences: actor → command → aggregate → event ← policy | sticky colours on the page follow this grammar |
| **Hotspot** | A spot that is unclear or disputed, recorded as an open question | |

## Status and reference codes

| Symbol | Meaning |
|---|---|
| `observed` | exists in code today |
| `target` | agreed goal, not yet coded |
| `proposed` | suggested, not agreed |
| `unknown` | not yet known |
| `accepted` / `rejected` / `superseded` | decision approved / rejected / replaced by a later one (keep the original text; `supersededBy` points at the new D) |
| `open` / `resolved` / `deferred` | question open / answered / postponed |
| **D12** | Decision number 12 in `docs/domain-model.json` and `.context/decisions/` |
| **OQ8** | Open question number 8 |

**Rule for mentioning D/OQ:** every time a code appears in speech, a log or the model, add its
short content right after it: "D15 (the official price changes only through an approved review)",
"OQ41 (who owns the venue list when Quant needs a new exchange)". The reader never has to look it
up. The renderer annotates codes on the page; agents must do it themselves when they speak.

## Writing style for the model and the pages

Standard sentence: **"A setup becomes active only after price enters its entry range."** One
sentence, one idea, a subject, what happens and when. Applies everywhere there is text:
description, invariant, rule, transition, event, use case, building block purpose, relation.

- Subject first: *The guardrail rejects the setup*, *The Quant Engine publishes the profile*,
  *The gateway fans out the tick*. Avoid the passive when someone does the act.
- Keep each sentence under about 20 words. A second idea gets a new sentence. No long dashes or
  semicolons as joiners; no parenthesis that holds a whole clause.
- Say the business act, not the code: *"the setup moves to active"* instead of *"UPDATE status
  = ACTIVE"*. Field names and function names appear only when the reader must go find that exact place.
- A D/OQ code always carries its short content; the renderer annotates it on the page, agents do
  it in speech.
- Use the DDD terms from the tables above. Use the business words with the meaning in the
  "Business language" table; do not coin a new word for something that already has a name.
- Transition: who acts and on what condition; a side effect states the business consequence
  (*"the setup leaves the radar feed"*), not a technical command.
- Event: write it as something that just happened, with a subject, so another context knows what
  to react to. Name it `<context>.<aggregate>.<past-tense>` (for example `radar.setup.activated`).
- Command: imperative (`confirm`, `invalidate`). Do not translate or alias command and event names.
