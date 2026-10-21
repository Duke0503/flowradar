# Data Contracts

All contracts below come from the design conversation. They are drafts. The final field names
belong to the domain model and the Protobuf definitions.

## OrderFlowBar

The standard shape between data and chart. It comes from `lightweight-orderflow-charts`. One bar is
one candle plus the buy and sell volume at every price level inside it. The helper
`clusterOrderFlowBarsByMintick` groups levels into the smallest price step and fills empty levels,
so footprint cells line up with the candle at any zoom.

A bar carries the time, open, high, low and close, and a `levels` array. Each level has a price,
`bidVolume`, `askVolume` and `delta`. Optional fields per bar are `vwap`, `pocPrice`, `deltaMin` and
`deltaMax`. These names come from the research report, not from the conversation.

Note: the report describes `bidVolume` as aggressive buy volume and `askVolume` as aggressive sell
volume. That is the reverse of the usual meaning (a buy aggressor lifts the ask). Check the
library's own definition before the gateway fills these fields.

## Redis message envelope

Every message through Redis uses this wrapper.

| Field | Meaning | Example |
|---|---|---|
| `topic` | What the message is about | `orderflow:btcusdt:1m`, `radar:signal:new` |
| `scope` | Who may receive it | not defined in the source |
| `payload` | The data itself, packed with Protobuf and Zstd | |

The sources also use `channel:btc_trades` and `channel:ai_signals`. See OQ4 in [../known-issues.md](../known-issues.md).

## State vector (sent to the AI)

A compact JSON snapshot of about 400 to 500 tokens. Intraday (15m) and HTF (4H and 1D) data are kept
apart.

```json
{
  "symbol": "BTCUSDT",
  "timeframe": "15m",
  "market_bias_htf": "BULLISH_CONTINUATION",
  "current_price": 64250.0,
  "tpo_levels": { "vah": 64800.0, "poc": 64100.0, "val": 63650.0 },
  "order_flow_indicators": {
    "cvd_divergence": "BULLISH_ABSORPTION_AT_POC",
    "open_interest_delta_1h": "+8.4%",
    "funding_rate": 0.0085,
    "liquidation_magnets": {
      "short_liquidation_pool": 65400.0,
      "long_liquidation_pool": 63200.0
    }
  }
}
```

In the improved flow, the prompt also carries a menu of anchor levels, the three most similar past
setups with outcomes, the win rate of the last 20 setups and five post-mortem lessons.

## AI output (trade setup)

The AI must return an object that Pydantic validates. The draft in the conversation:

```json
{
  "status": "OPPORTUNITY_FOUND",
  "direction": "LONG",
  "confidence": 0.88,
  "rationale": "Price retests the POC at 64100 and shows strong sell-side absorption with a rising CVD divergence.",
  "trade_setup": {
    "trigger_condition": "A 5m candle closes above 64300 with positive volume delta",
    "invalidation_condition": "Price breaks and holds below the VAL at 63650 before the order triggers",
    "entry_range": [64150.0, 64300.0],
    "stop_loss": 63550.0,
    "take_profit_targets": [64800.0, 65400.0],
    "risk_reward_ratio": 2.2
  }
}
```

In the accuracy pipeline, the fields are `trigger_condition`, `invalidation_condition`,
`entry_price`, `stop_loss`, `tp1`, `tp2`, `tp3`. The two shapes differ (an entry range or a single
entry price, a target list or three named targets). That needs one decision.

The guardrail recomputes `risk_reward_ratio` itself and ignores the AI's value. See
[../ai-accuracy-pipeline.md](../ai-accuracy-pipeline.md).

## Setup status events

Status changes go through Redis and then WebSocket, as JSON events. The states are
`PENDING_TRIGGER`, `ACTIVE`, `HIT_TP`, `HIT_SL`, `INVALIDATED`, and `EXPIRED` (added by D2). The
frontend writes each event into the TanStack Query cache under the key `['ai-radar-signals']`.

## Time

All timestamps are UTC Unix milliseconds in the backend and database. Only the browser converts to
local time.
