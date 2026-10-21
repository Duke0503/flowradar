# Known Issues and Conflicts

## System bottlenecks and their solutions

| # | Issue | Cause | Solution |
|---|---|---|---|
| 1 | CoinGlass rate limit runs out | If every browser calls CoinGlass, 500 users exhaust the quota in minutes | One background worker polls CoinGlass on a schedule and writes to Redis. Browsers read only from the gateway |
| 2 | TPO and footprint calculation lags | Building letters and price-level volume from 30 days of ticks on demand takes seconds | Pre-aggregate 1m and 15m blocks into TimescaleDB as candles close. Serve blocks, not raw ticks |
| 3 | The AI invents price levels | An LLM may suggest an entry at $62,000 when BTC is at $75,000 | Pass a menu of computed anchor levels and force entries, stops and targets onto them |
| 4 | Intraday and HTF signals disagree | A 15m scan says short while the 4H scan says strong long | Inject the HTF bias into the intraday prompt. A counter-trend setup needs higher confidence and tighter targets |
| 5 | Zombie signals | A setup from two hours ago is useless if price already passed its target or stop | Give every setup a lifecycle and update status from live price |

## Conflicts found between the source documents

These were found while reading both sources. They are recorded as open questions for the
domain discussion. None of them is decided.

### OQ1: What is the minimum R:R?

- The design conversation says the guardrail needs R:R of at least 1:2.0 for every setup, and a
  later section says 2.0 for intraday and 3.0 for HTF.
- The research report says a setup with R:R below 3.0 is rejected.
- Owner: product. The value must be one number per timeframe, set in one place.

### OQ2: Does "tighten targets" break the R:R gate?

The HTF-bias rule says a counter-trend intraday setup should "tighten Take Profit to the nearest
POC". Closer targets lower R:R. A setup can pass at generation time and then be rejected by the
guardrail for the very change the rule asked for. The order of the two rules needs to be defined.

### OQ3: Who computes the value area, and where is the single copy?

The source design has Go collecting ticks and Python computing TPO, but the chart also draws POC,
VAH and VAL. D3 says Python is the only computer. The open part is how Python publishes the
result to Go and the browser so the chart never recomputes it.

### OQ4: Which Redis channel names are the standard?

The documents use `channel:btc_trades`, `channel:ai_signals`, `orderflow:btcusdt:1m` and
`radar:signal:new`. Pick one naming rule and one envelope.

### OQ5: The sample code computes CVD per candle, not across candles

The first Python draft partitions the cumulative sum by `candle_time`, so CVD restarts at every
candle. A true CVD runs across candles. The draft's `layer_3` also takes swing lows as input but
the source does not say who detects the swing points. This is draft code, not a decision.

### OQ6: Is the 2% drift limit a rule or a guess?

The drift guard invalidates a setup that moves more than 2% from entry before it triggers. A flat
2% is large for a 15m scalp and small for a 4H swing. The value probably depends on timeframe.

### OQ7: How long may a setup stay pending before it expires?

D2 introduces the `EXPIRED` state. The time limit per timeframe is not set anywhere.

## Commercial release questions

OQ8 to OQ13 cover data rights, signal regulation and code licenses. They are explained in
[commercialization.md](commercialization.md) and [licensing.md](licensing.md).

## Language note

The source documents mix Vietnamese and English. The project is now English only. The code in
the PDF contains Vietnamese comments and messages. Rewrite them in English when the code moves
into the repositories.
