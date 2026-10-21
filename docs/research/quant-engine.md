# Quant Engine

A radar that relies on slow indicators such as RSI, SMA or MACD has no edge. The edge comes from
reading volume asymmetry and the footprint of smart money through order flow. Microstructure data
has very high resolution. It lets the AI see small changes in buy and sell pressure before price
moves.

Per decision D3, the Python Quant Engine is the only place that computes POC, VAH, VAL and CVD.

## Value area and volume profile

Market Profile and TPO analyse volume distribution over horizontal price levels within a period,
not along the time axis [4].

- POC (Point of Control): the price level with the highest traded volume in the session.
- Value Area: the zone that holds about 70% of the session's total volume [21]. It marks the
  short-term balance boundaries of the market.

### Algorithm, step by step

The algorithm comes from open-source community code, including PineScript samples [22]. It runs in
Python or Rust.

Note: the original formulas were lost in the source text. The notation below is reconstructed from
the described steps and the two-pointer method.

1. Bucket all volume of the time frame into price bins (a price histogram). Compute the session
   total volume, `V_total`.
2. Find the bin with the highest volume. Its index is the anchor, `i_poc`.
3. Start the accumulated volume at `V_acc = V[i_poc]`. Set two pointers, `i_up = i_poc` and
   `i_dn = i_poc`. The upper pointer moves to higher prices and finds the Value Area High (VAH).
   The lower pointer moves to lower prices and finds the Value Area Low (VAL).
4. At each iteration, compare the volume one step above, `V_up = V[i_up + 1]`, with the volume one
   step below, `V_dn = V[i_dn - 1]`. The side with more volume is added to `V_acc`. Its pointer
   moves one step outward [22].
5. Repeat until `V_acc >= 0.70 * V_total` [22].
6. VAH is the price at `i_up`. VAL is the price at `i_dn`.

Note: the source does not state how ties or the double-step variant are handled. Use the
description above as written.

### wickra

Repository: `wickra-lib/wickra` (Rust) [21].

- Implements the full value area logic.
- Offers batch processing of arrays for history and streaming, tick by tick, for live data.
- Lets the radar update VAH, VAL, POC, Initial Balance and volume gaps (Single Prints) with
  minimal latency and no bottleneck on the real-time flow.

## CVD divergence detection

CVD is the running total of the difference between aggressive buying and aggressive selling
(Market Buy Volume minus Market Sell Volume). CVD divergence is one of the strongest reversal
signals in order flow trading. It occurs when the price trend and the market-order pressure trend
move in opposite directions [4].

Projects with divergence logic: `mahmoud20138/OrderFlow-Analysis-Pro` and the trading bot
`ymys/Bot-Auto-Screening-Bybit-trading` (the report names its code `bybit_bot_v8`) [23]. The radar can use the
static scoring of `bybit_bot_v8` as a base score before any machine learning step.

### Detection steps [24]

1. Continuously find new price swing highs and lows (market structure).
2. Compare them with the matching highs and lows of the in-memory CVD series.
3. If price makes a Lower Low and CVD makes a Higher Low, record a Regular Bullish Divergence.
   Selling pressure is exhausted, and the fall comes from withdrawn passive buy orders
   (a liquidity void). The market is ready to reverse [4].
4. Check the signal against Smart Money Concepts context: Order Blocks, Change of Character
   (CHoCH) and Break of Structure (BOS).
5. Drop signals inside trap zones (Trap Detection), such as Dead Cat Bounces, and bullish
   exhaustion signals with RSI above 75.

The bearish case is the mirror image. The report describes only the bullish case.

### Pre-scoring rules [24]

| Condition | Effect |
|---|---|
| Relative volume (RVOL) above 5.0 ("Nuclear RVOL") or a safe funding rate | +1 point |
| Confirmed CVD divergence | +2 or -2 points |
| RVOL below 2.0 | Reject the signal as a fakeout (Fakeout Protection) |

Note: the report cites `bybit_bot_v8` without a repository. Matching it to the `ymys` project is an inference from the shared scoring logic.

Note: the source says "+1" for RVOL or funding and "+/- 2" for divergence. The sign rule for
divergence is not explained.

### OBV divergence

`FelixWayne0318/AlgVex_Mechanical` adds On-Balance Volume (OBV) divergence smoothed by a 20-period
EMA [27]. It gives a macro volume flow view that confirms the micro flow from CVD.

## Machine learning in quant trading

Repository: `stefan-jansen/machine-learning-for-trading` [28]. It is an academic method for
feature engineering from order flow data.

- Five feature families: momentum, reversal, volatility, liquidity, microstructure.
- Triple-barrier labelling.
- Deflated Sharpe Ratio, to control multiple testing and avoid overfitting from excessive backtests.
- Gradient boosting models such as LightGBM, or TabPFN, for tabular forecasting.
- SHAP values to explain model decisions, before passing results to the multi-agent AI radar.

## Other indicator projects

| Project | Source number | What the report says |
|---|---|---|
| `srlcarlg/srl-python-indicators` | [20] | Listed only. Title: Order Flow Ticks, Volume |
| `SpiralDevelopment/RSI-divergence-detector` | [25] | Listed only |
| `Mykola-Quant/retail-crypto-alpha` | [26] | Listed only |
| `mahmoud20138/OrderFlow-Analysis-Pro` | [23] | Divergence logic, absorption and sweep rules (see [../open-source-research.md](../open-source-research.md)) |
| `ymys/Bot-Auto-Screening-Bybit-trading` | [24] | Scoring, RVOL fakeout filter, trap filter |

Note: the numbers 20, 25 and 26 appear in the sources but are not attached to any sentence in the report.

## Placement in FlowRadar

- Value area: two-pointer method, computed in Python (D3), published to Redis for Go to forward.
- CVD divergence: price swing points compared with CVD swing points.
- The deterministic score is computed before any model call. See [ai-radar.md](ai-radar.md).
