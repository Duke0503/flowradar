# Presentation Layer

The frontend of an order flow site must show per-price traded volume (footprint), orderbook
heatmaps and volume profiles without freezing the browser. Building a custom rendering engine for
this is costly and slow when millions of data points update each second. The research therefore
separates the data stream from the drawing system.

## lightweight-orderflow-charts

Repository: `awahab07/lightweight-orderflow-charts` [1]. Also see the TradingView discussion [2].

It is a broker-neutral rendering layer. It targets web apps that already own a market data
collector and need a drawing surface. It plugs into TradingView `lightweight-charts`, so the
chart logic is not rewritten.

### OrderFlowBar contract

Raw exchange data is normalised into one strict JSON structure [1].

| Field | Meaning |
|---|---|
| time | Bar time |
| open, high, low, close | OHLC prices |
| levels | Array with one entry per price level |
| levels[].bidVolume | Aggressive buy volume at the level |
| levels[].askVolume | Aggressive sell volume at the level |
| levels[].delta | Delta at the level |
| vwap (optional) | Volume weighted average price |
| pocPrice (optional) | Point of Control price |
| deltaMin, deltaMax (optional) | Delta extremes of the bar |

Note: the report calls `bidVolume` aggressive buy volume and `askVolume` aggressive sell volume,
as written. Check the field semantics against the library before mapping exchange data.

The contract fully separates the API layer from the drawing layer. The frontend then focuses on
display only.

### Rendering features [1]

- Footprint cells shaded by volume scale (scale-driven shading).
- Imbalance highlighting between buyers and sellers, with a configurable percentage.
- POC marking through border and thickness properties.
- Delta Summary subchart under the footprint, with a custom colour scale per row. It shows the
  overall buy and sell pressure of each candle.

### Dual-surface formatting

Price axis precision and on-cell volume shortening are configured separately [1].

| Surface | Options |
|---|---|
| Price axis | `priceDecimalDigits`, `priceRounding` |
| Volume on cells | `valueIntegerDigits`, `valueDecimalDigits`, `valueUnitVisible` |

Example: the price axis shows "50,245.50" while the cell volume shows "12.3k".

### Mintick bucketing

Helpers such as `clusterOrderFlowBarsByMintick` group price levels into mintick buckets [1].

- They keep the price grid uniform and evenly spaced.
- They cope with sparse data, where the exchange sends nothing for levels with zero volume.
- They keep footprint rows geometrically fixed when the user zooms.

## kline-orderbook-chart

Repository: `tapedelta/kline-orderbook-chart` [5].

- Overlays a Liquidation Heatmap and the orderbook as a background on the candle chart.
- Shows concentrations of leveraged positions, so users can see zones of possible cascading liquidations.
- Draws large whale order bubbles for liquidity shocks.
- Includes Visible Range Volume Profile (VRVP) and TPO.
- Exposes an API to add custom indicators.

## aggr

Repository: `Tucsky/aggr` [6].

- Vue.js with a multi-stream WebSocket interface. It aggregates and shows live trades from many exchanges.
- Timeout-based aggregation merges trades with the same timestamp to remove noise.
- Rolling sums are computed over fixed periods.
- The result is a compact trade tape.
- Dynamic audio plays different tones by trade size and volume.
- Rate limiting of the display handles tens of thousands of trade streams per second.

## flowsurface-rs

Repository: `flowsurface-rs/flowsurface` [9] [10]. It is a Rust multi-window desktop app, not a web app.

- Renders Footprint and Heatmap.
- Supports tick-count aggregation (TickBased klines) instead of time frames. Switching from time
  to tick count re-aggregates the whole history at once, without a new server load [10].
- Loads history from the Binance REST API, or from an HTTP server that serves Apache Arrow IPC.
  This moves large columnar data to the client very fast [9].

## mplchart

Repository: `furechan/mplchart` [11].

- Python (Pandas or Polars) charting of candlesticks, volume, SMA, MACD and overbought/oversold lines.
- Fits server-side rendering, so the radar can produce static images for a multimodal AI.
- See [ai-radar.md](ai-radar.md) for how the image is used.

## Tealstreet

Listed in the sources only [3]. The report does not discuss it.

## Project comparison

| Project | Language / framework | Core visual features | Architecture value |
|---|---|---|---|
| `lightweight-orderflow-charts` | TypeScript / React | Footprint, Delta Summary, dual-surface formatting, POC | Provides the `OrderFlowBar` structure and price grid handling through mintick bucketing |
| `kline-orderbook-chart` | JavaScript / HTML5 | Orderbook heatmap, Liquidation Heatmap, VRVP, TPO | Shows the orderbook behind the chart and leverage liquidation clusters |
| `aggr` | Vue.js / TypeScript | Tape noise filtering, merge by timestamp, dynamic audio | Handles tens of thousands of trade streams per second with display rate limiting |
| `flowsurface-rs` | Rust | Multi-window management, TickBased klines, Arrow IPC stream | Flexible resource management and trade aggregation independent of time |
| `mplchart` | Python / Polars | Static charts, bands, support and resistance trend lines | Makes market screenshots for vision AI models |

## FlowRadar mapping

- Use `lightweight-orderflow-charts` on the existing TradingView Lightweight Charts v5.
- Take only the liquidation overlay idea from `kline-orderbook-chart`.
- Take the batching idea from `aggr`: group same-millisecond ticks before drawing, in a Web Worker.
- `flowsurface-rs` is reference only for TPO math and Arrow IPC. It is a desktop app.
