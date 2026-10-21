# Quant Engine Draft (layers 1 to 3)

The design conversation ended with a first Python class, `OrderFlowQuantEngine`. It covers layers
1 to 3 of the accuracy pipeline in [../ai-accuracy-pipeline.md](../ai-accuracy-pipeline.md). This page
keeps the draft as a starting point and lists what is wrong or missing. It is not production code.

Comments and messages are rewritten in English. One line of the original SQL was cut off by the
PDF layout. The `candle_time` expression below is reconstructed from the surrounding text and must
be checked.

## What each layer does

| Layer | Method | Input | Output |
|---|---|---|---|
| 1 | `layer_1_process_footprint_and_cvd` | Raw trades in DuckDB | A footprint table per candle and price bin, with buy, sell, total volume and delta |
| 2 | `layer_2_calculate_value_area` | The layer 1 table | POC, VAL, VAH and total volume, by the two-pointer method at 70% |
| 3 | `layer_3_deterministic_prescore` | Price lows, CVD lows, RVOL | `REJECTED`, `TRIGGER_AI` or `HOLD`, with a score and reasons |

## The draft

```python
import duckdb
import numpy as np
import pandas as pd


class OrderFlowQuantEngine:
    def __init__(self):
        # DuckDB runs fully in memory.
        self.conn = duckdb.connect(database=':memory:')
        self._init_schema()

    def _init_schema(self):
        """Create the table for raw trades."""
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS raw_trades (
                timestamp TIMESTAMP,
                price DOUBLE,
                size DOUBLE,
                side VARCHAR   -- 'BUY' or 'SELL' (the aggressor side)
            );
        """)

    def ingest_trades(self, df_trades: pd.DataFrame):
        """Load raw trades into DuckDB."""
        self.conn.register('df_view', df_trades)
        self.conn.execute("""
            INSERT INTO raw_trades
            SELECT CAST(timestamp AS TIMESTAMP), price, size, side
            FROM df_view;
        """)
        self.conn.unregister('df_view')

    # Layer 1: in-memory processing with DuckDB (footprint and CVD)
    def layer_1_process_footprint_and_cvd(self, timeframe_minutes: int = 15, mintick: float = 1.0):
        """Group raw trades into footprint bars and compute delta and CVD."""
        query = f"""
            WITH binned_trades AS (
                SELECT
                    -- Group by timeframe, for example 15 minutes (reconstructed expression)
                    date_trunc('minute', timestamp)
                      - INTERVAL (extract(minute from timestamp) % {timeframe_minutes}) MINUTE
                      AS candle_time,
                    -- Group price by the minimum price step (mintick bucketing)
                    ROUND(price / {mintick}, 0) * {mintick} AS price_bin,
                    SUM(CASE WHEN side = 'BUY'  THEN size ELSE 0 END) AS buy_volume,
                    SUM(CASE WHEN side = 'SELL' THEN size ELSE 0 END) AS sell_volume
                FROM raw_trades
                GROUP BY 1, 2
            ),
            delta_calc AS (
                SELECT
                    candle_time, price_bin, buy_volume, sell_volume,
                    (buy_volume + sell_volume) AS total_volume,
                    (buy_volume - sell_volume) AS volume_delta
                FROM binned_trades
            )
            SELECT
                candle_time, price_bin, buy_volume, sell_volume, total_volume, volume_delta,
                -- "CVD" as drafted: a running sum inside each candle, ordered by price
                SUM(volume_delta) OVER (
                    PARTITION BY candle_time
                    ORDER BY price_bin
                    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
                ) AS cvd
            FROM delta_calc
            ORDER BY candle_time ASC, price_bin ASC;
        """
        return self.conn.execute(query).fetchdf()

    # Layer 2: two-pointer value area (POC, VAH, VAL)
    def layer_2_calculate_value_area(self, footprint_df: pd.DataFrame):
        """POC and value area from the two-pointer method, at 70% of total volume."""
        if footprint_df.empty:
            return None

        # Total volume at each price level across the whole input.
        price_profile = (
            footprint_df.groupby('price_bin')['total_volume'].sum().reset_index()
        )
        if price_profile.empty:
            return None

        # Sort by volume to find the POC, the level with the most volume.
        by_volume = price_profile.sort_values(by='total_volume', ascending=False).reset_index(drop=True)
        poc_price = by_volume.loc[0, 'price_bin']
        total_volume = by_volume['total_volume'].sum()
        target_value_area_vol = total_volume * 0.70

        # Sort by price for the two-pointer scan.
        df_sorted = price_profile.sort_values(by='price_bin').reset_index(drop=True)
        poc_idx = df_sorted[df_sorted['price_bin'] == poc_price].index[0]

        up_idx = poc_idx
        down_idx = poc_idx
        accumulated_vol = df_sorted.loc[poc_idx, 'total_volume']

        while accumulated_vol < target_value_area_vol:
            vol_up = (df_sorted.loc[up_idx + 1, 'total_volume']
                      if (up_idx + 1) < len(df_sorted) else -1)
            vol_down = (df_sorted.loc[down_idx - 1, 'total_volume']
                        if (down_idx - 1) >= 0 else -1)
            # Grow toward the side with more volume.
            if vol_up >= vol_down and vol_up != -1:
                up_idx += 1
                accumulated_vol += df_sorted.loc[up_idx, 'total_volume']
            elif vol_down != -1:
                down_idx -= 1
                accumulated_vol += df_sorted.loc[down_idx, 'total_volume']
            else:
                break

        return {
            'POC': float(poc_price),
            'VAL': float(df_sorted.loc[down_idx, 'price_bin']),
            'VAH': float(df_sorted.loc[up_idx, 'price_bin']),
            'TotalVolume': float(total_volume),
        }

    # Layer 3: deterministic pre-score and CVD divergence
    def layer_3_deterministic_prescore(self, price_lows: list, cvd_lows: list, rvol: float):
        """Hard scoring filter. Decides whether the AI may be called."""
        score = 0
        signals = []

        # 1. Relative volume filter against fakeouts.
        if rvol < 2.0:
            return {
                'status': 'REJECTED',
                'reason': f'RVOL too low ({rvol:.2f} < 2.0): risk of a fakeout',
                'score': 0,
            }
        elif rvol >= 5.0:
            score += 1
            signals.append('Nuclear RVOL spike (+1)')

        # 2. Bullish divergence: price makes a lower low, CVD makes a higher low.
        if len(price_lows) >= 2 and len(cvd_lows) >= 2:
            price_lower_low = price_lows[-1] < price_lows[-2]
            cvd_higher_low = cvd_lows[-1] > cvd_lows[-2]
            if price_lower_low and cvd_higher_low:
                score += 2
                signals.append('Bullish CVD divergence, absorption (+2)')

        # 3. Threshold to call the AI.
        if score >= 2:
            return {
                'status': 'TRIGGER_AI', 'score': score, 'signals': signals,
                'reason': 'Quant threshold met. Build the state vector and call DeepSeek.',
            }
        return {
            'status': 'HOLD', 'score': score, 'signals': signals,
            'reason': 'Noise, or the score is too low. Skip the AI call to save tokens.',
        }
```

## Defects and gaps

| # | Where | Problem | Fix to decide |
|---|---|---|---|
| 1 | Layer 1, `cvd` | The sum is partitioned by candle and ordered by price. That is a running sum over price levels, not over time. True CVD is a running sum of each candle's total delta across candles (OQ5) | Compute one delta per candle, then a window sum ordered by `candle_time` |
| 2 | Layer 1 | The query returns all candles, but layer 2 groups every candle together. The value area then covers the whole input, not one session | Call layer 2 per session, or per candle window, and define the session |
| 3 | Layer 1 | `candle_time` expression was cut in the source and is reconstructed. It uses minute modulo the timeframe, so it is wrong for timeframes over 60 minutes such as 4H | Use `time_bucket` or `date_bin` |
| 4 | Layer 2 | POC is found by sorting, then the sorted frame is re-sorted. A tie for the largest volume picks the first one | Decide the tie rule |
| 5 | Layer 2 | A standard value area expands by two levels at once, comparing the sum of the next two above to the next two below. This draft adds one level at a time | Decide the exact method and test it against a known profile |
| 6 | Layer 3 | RVOL below 2.0 rejects every candidate. The pipeline says to reject only a breakout with low RVOL | Check whether it is a breakout first |
| 7 | Layer 3 | Only bullish divergence exists. There is no bearish side, no smoothing (EMA 20 of CVD and OBV) and no trap filter (RSI above 75, crowded funding) | Add them to match the pipeline |
| 8 | Layer 3 | `price_lows` and `cvd_lows` are inputs. Nothing detects the swing points | Add a swing detector, with parameters |
| 9 | All | `side` assumes the exchange gives the aggressor. Binance `aggTrade` gives it through the maker flag | Map it in the venue adapter |
| 10 | All | The class loads trades through a DataFrame. The real flow reads from Redis | Add the ingest path |
| 11 | All | No retention job. The pipeline says to delete data older than 24 hours every hour | Add it |
| 12 | All | No tests | Add tests with a known profile |

Items 1, 6 and 8 touch the open questions in [../known-issues.md](../known-issues.md).
