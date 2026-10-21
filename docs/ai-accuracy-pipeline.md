# AI Accuracy Pipeline

Rule of the whole pipeline: the AI never does arithmetic and never invents a price. Code
computes the numbers. The AI only reasons over them and proposes a setup.

Every candidate passes five layers in order. Most candidates die before the AI is called.

## Layer 1: In-memory computation with DuckDB

- Python loads the last 24 hours of trades from Redis into an in-memory DuckDB table.
- One query groups ticks by timeframe (15m, 1H, 4H) and by price step (the mintick bucket) into
  OrderFlowBars with buy volume, sell volume, total volume and delta per level.
- An hourly job deletes data older than 24 hours so memory stays small.
- Pattern from `cryexc-backend`.

## Layer 2: POC, VAH and VAL by the two-pointer method

1. Find the price level with the largest volume. That is the POC. Compute the total volume.
2. Put one pointer just above the POC and one just below.
3. Compare the volume at the two pointers. Add the larger one to a running total and move that
   pointer one level outward.
4. Repeat until the running total reaches 70% of total volume.
5. The final pointer positions are VAH (top) and VAL (bottom).

Pattern from `wickra` and PineScript TPO implementations.

## Layer 3: Deterministic pre-score and trap filter

The pre-score decides whether the AI is called. It removes about 80% of noisy signals.

| Check | Effect |
|---|---|
| Price makes a lower low while CVD makes a higher low (bullish divergence), using smoothed CVD and OBV | +2 |
| RVOL below 2.0 during a breakout | Reject at once, do not call the AI |
| RVOL above 5.0 | +1 |
| Long candidate while RSI is above 75 or funding is crowded long | Reject as an exhaustion trap |

A candidate needs a score of at least 2 to reach the AI. Pattern from `Bot-Auto-Screening-Bybit`.

## Layer 4: Memory and claim validation

When a candidate passes layer 3, the system builds the State Vector and enriches the prompt.

- **Retrieval.** Python embeds the current order flow structure and searches ChromaDB for the
  three most similar past setups and their outcomes. It adds the win rate of the last 20 setups
  and five post-mortem lessons from stopped-out setups.
- **Claim validation.** After the AI answers, code checks its claims against measured data. If the
  AI says "very strong trend" while ADX is low, confidence is lowered automatically.

Pattern from `LLM_trader`.

## Layer 5: Schema lock and lifecycle guard

- **Schema.** DeepSeek must return a Pydantic-validated object with `trigger_condition`,
  `invalidation_condition`, entry range, `stop_loss`, take-profit targets and `risk_reward_ratio`.
  A malformed answer is rejected.
- **Anchor levels.** The prompt offers a menu of verified levels (VAL, POC, VAH, liquidation
  clusters). Entries, stops and targets must sit on them.
- **R:R check.** Code recomputes R:R as `|TP - Entry| / |Entry - SL|`. Intraday setups need at
  least 2.0. HTF setups need at least 3.0 in the pipeline description. See OQ1 in
  [known-issues.md](known-issues.md).
- **Stop placement.** The stop must lie beyond the TPO or liquidation barrier.
- **HTF bias.** The latest 4H and 1D bias is injected into every intraday prompt. A counter-trend
  intraday setup needs higher confidence and tighter targets.

Pattern from `crypto-signals` and `Auto-Crypto-Tradingjournal`.

## TradeSetup lifecycle

```
PENDING_TRIGGER --> ACTIVE --> HIT_TP
        |             |  \--> HIT_SL
        |             \------> (TP1 hit: stop moves to break-even)
        +--> INVALIDATED
        +--> EXPIRED
```

| State | Meaning |
|---|---|
| `PENDING_TRIGGER` | The guardrail accepted the setup. Price has not yet entered the entry range |
| `ACTIVE` | Price entered the entry range. This is not a claim that any user took the trade (D1) |
| `HIT_TP` / `HIT_SL` | The setup ended at a take-profit or at the stop loss |
| `INVALIDATED` | Price hit the invalidation condition, or drifted more than 2% from entry, before it triggered |
| `EXPIRED` | The setup timed out before it triggered. This state is new in the domain model; the source documents do not have it |

Rules around the lifecycle:

- Take-profit is a ladder: TP1 40%, TP2 40%, TP3 20%. When TP1 is hit, the stop moves to
  break-even plus a 0.25% fee buffer, and the UI says so.
- Outcome is recorded as an R-multiple, for example +2.1R or -1R, not only win or loss (D2).
- Win rate counts only setups that became active and then ended at a take-profit. Invalidated and
  expired setups are tracked in a separate invalidation rate (D2).
- The Python service is the only writer of status changes (D3). Go publishes a throttled price
  channel and bar-close events, and Python compares price to the setup's levels.
- A rejected setup is stored with its rejection reason and is never shown to users (D5).

## Event-driven trigger

The AI is called only when one of these happens, not on a fixed timer:

- A 15m or 4H candle closes.
- The quant engine detects an anomaly: a liquidation spike above about $5M, or a CVD divergence
  at the POC.

## State vector

The State Vector is a compact JSON snapshot of about 500 tokens. It holds the symbol, timeframe,
HTF bias, current price, TPO levels (VAH, POC, VAL), order flow signals (CVD divergence, OI change,
funding), and liquidation magnets. Intraday (15m) data and HTF (4H and 1D) data are kept apart.
