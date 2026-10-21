# Execution and Risk

> **Deferred.** Per decision D1 (FlowRadar publishes signals only), this layer is out of the MVP. Kept as reference. The signal lifecycle, multi-TP ladder and entry-drift ideas carry over as signal tracking, not as orders.

Once the AI radar confirms a signal, the best open-source projects do not just send a blind market
order. They run a Managed Trade Lifecycle protected by several guardrails. The AI never crosses
a calculated financial risk limit.

In FlowRadar, "order" below means a tracked TradeSetup. See D1 and D3: the Python Opportunity Radar
owns the setup lifecycle, and ACTIVE means price entered the entry range.

## Bracket orders

Project: `lagarcess/crypto-signals` [35].

- An entry order always carries its Take-Profit and Stop-Loss atomically, in broker API calls.

## Risk:Reward validation

Projects: `bybit_bot_v8` and `LLM_trader` [24].

- Every LLM trade idea must pass a Risk:Reward (R:R) check.
- Expected profit is measured against the stop loss, using Fibonacci retracement or a microstructure range.
- If the AI proposes a low R:R (example: below 3.0), code-enforced logic rejects the order.
  Code cannot be overridden, however high the AI confidence score is.

## Risk-based position sizing

```
qty = RISK_PER_TRADE / |entry - stop|
```

- The potential loss is the same for every position, whatever the asset price [35].

## Slippage and entry-drift guard

Project: `anvilfilbert/Auto-Crypto-Tradingjournal` [39].

- The guard compares the real fill price on the exchange with the target price computed by the
  radar at signal time.
- If the drift exceeds the tolerance (example: 2%), the system sends a cancel, or an immediate
  market-close, and rejects the trade.
- This stops chasing a price after news pushes the market too far.

## Multi-TP ladder

Same project [39].

| Level | Share of position |
|---|---|
| TP1 | 40% |
| TP2 | 40% |
| TP3 | 20% (kept for an extended move) |

- When TP1 fills, a reconciler moves the stop loss to break-even.
- The break-even stop includes a 0.25% buffer. It covers taker fees and exit slippage.
- The report says an "Opus" agent can set the split structure. It gives 40/40/20 as an example.

## schwagent

Project: `sbauwow/schwagent` [40].

- Capital is split into 5 parallel tranches. This handles T+1 settlement delay.
- A global Dry Run switch keeps the whole system in paper trading, with no real funds, until the user approves.

## bybit-py-mcp

Project: `BCusack/bybit-py-mcp` [37] [41]. It is a Model Context Protocol (MCP) server for Bybit.

- MCP is an abstraction layer that separates privileges. Every exchange action maps to strict
  Pydantic models. Examples: switching cross or isolated margin, setting leverage, configuring
  auto-add margin, querying Open Interest, reading Closed PnL history [41].
- It blocks hallucinated parameters, such as a 1000x leverage order or a negative stop price.
- It converts types automatically. Example: the `seq` field changes from integer to string to
  fix an inconsistency in the Bybit API.
- Any attempt by the agent to leave the defined structure raises a schema validation error.

## CryptoSignalApp patterns

Project: `onurcangnc/CryptoSignalApp` [42].

| Pattern or tool | Use |
|---|---|
| Proxy | A Redis client with lazy loading and connection pooling |
| Facade | Hides SQLite plus Redis behind one consistent interface |
| DCA Calculator | Average price tool for risk management |
| Exit Strategy Backtest Engine | Tests stop-loss strategies using ATR (Average True Range) to measure volatility |

## What carries over to FlowRadar

| Idea | FlowRadar use |
|---|---|
| Managed lifecycle | TradeSetup states, tracked by the Opportunity Radar |
| Multi-TP ladder 40/40/20 | Several target levels per signal, each with a hit status |
| Break-even move with 0.25% buffer | Optional stop adjustment shown on the signal |
| Entry-drift guard | Mark a setup invalid or expired when price runs past the entry range |
| R:R validation in code | Reject low R:R setups before publishing |
| Pydantic schemas | Keep as the schema contract for `TradeSetup` |
| Order placement, sizing, API keys, MCP exchange server, dry-run | Not used (D1) |
