# AI Radar

The quant layer produces hundreds of signals per hour. Most are traps caused by noise. The AI
Opportunity Radar filters them. Modern designs go beyond simple prompts. They use multi-agent
orchestration, vision analysis and Retrieval-Augmented Generation (RAG).

See also [../ai-accuracy-pipeline.md](../ai-accuracy-pipeline.md) for the FlowRadar pipeline.

## Multimodal flow and DeepSeek integration

Reference: `qrak/LLM_trader` ("Semantic Signal LLM") [29].

Flow:

1. The order flow system detects a possible confluence, for example a CVD divergence at the POC
   of the Value Area.
2. The radar captures a complete candlestick chart image with indicators. Format: PNG at 1920x1080.
3. The image is an extra input. It does not replace numbers. It goes in parallel with an array of
   quantitative indicator values.
4. Both are sent to a multimodal model.

Provider routing [29]:

- The official DeepSeek API (`deepseek-flash`, or a variant with good image recognition) is the
  primary route. The reasons given are native chart image input and strong reasoning.
- Multi-Provider AI Routing is the fallback when DeepSeek hits rate limits or an outage. It can
  switch to Google Gemini, OpenRouter, or local models through LM Studio or Ollama, as long as they
  accept image input.
- The same LLM-agnostic idea appears in `swarm-trader` (up to 13 LLM personas for market analysts)
  and `TradingAgents` [30]. It avoids vendor lock-in.

Claim Validation in `LLM_trader` [29]:

- Any LLM claim about trend strength is compared against the real ADX value.
- Any subjective rating of pattern quality is replaced by a deterministic scorer.
- This prevents hallucination.

Server-side chart images can come from `mplchart`. See [presentation-layer.md](presentation-layer.md).

## Independent opportunity scoring

Scoring runs in code before the model call. The rules are in [quant-engine.md](quant-engine.md):
CVD divergence points, RVOL threshold, trap filter and SMC context. The model scores the
opportunity from a deterministic base. It does not replace the math.

## Agentic RAG with a vector database

`LLM_trader` uses the ChromaDB vector database as long-term memory, the "Brain with Memory" [29].
Putting thousands of past trades in every prompt overflows the context window, reduces the AI's
focus and raises inference cost.

Instead, local semantic search (embedding model `bge-base-en-v1.5`) retrieves only the most
relevant items before each decision [29].

| Retrieved item | Purpose |
|---|---|
| The 3 past trades with the most similar market microstructure | Precedents for the current setup |
| Win rate and average P&L of the last 20 trades | Keeps the AI probabilistic and aware of capital management |
| The 3 sharpest trading rules, ranked by the "surprise ratio" of the Reflection Engine | Lets the AI discount lucky outcomes |
| Up to 5 post-mortem lessons from earlier bad decisions | Avoid repeated mistakes |

Token budget controls [29]:

| Control | Value |
|---|---|
| Ceiling per request | 100,000 estimated tokens |
| ChromaDB "brain" space | 50,000 tokens |
| Model safety margin | 10% |
| Reserved zone for chart image encoding | 16,384 tokens |

If retrieval exceeds the context limit, the system cuts low-priority blocks first, such as news
RAG data from CoinDesk or DeFiLlama. It never cuts the risk management instructions, the current
position and the response format.

Note: FlowRadar decision D2 says win rate counts only setups that became active. Apply that rule
when computing the "win rate of the last 20".

## Pydantic schema enforcement

An LLM that returns prose instead of a clear JSON object stalls any downstream system.

Solution: Python with Pydantic [32].

- Projects `agentic-rag`, `crypto-signals` and `LLM_trader` define the expected output as a
  Pydantic schema, for example `TradeSetup` or `ResearchPaperExtraction`.
- The schema defines required fields: `signal_direction` (Enum LONG/SHORT), `entry_price` (float),
  `stop_loss` (float), `take_profit_levels` (list of floats) and `confidence_score`.
- The JSON schema from Pydantic is sent in the API request to DeepSeek or OpenAI through Function
  Calling or the Structured Output API.
- The model is then constrained at the token-generation level. The radar always receives JSON
  valid in its types, with no pleasantries.
- Hidden reasoning and long text research reports go into a separate field. They do not pollute
  the signal JSON [29].

## Multi-agent and related projects

| Project | What the report says | Source |
|---|---|---|
| `qrak/LLM_trader` | Main reference: multimodal flow, ChromaDB RAG, claim validation, provider routing | [29] |
| `tauricresearch/tradingagents` | Multi-agent LLM financial trading framework. LLM-agnostic | [30] |
| `zhound420/swarm-trader` | Up to 13 LLM personas for analyst roles | [31] |
| `lagarcess/crypto-signals` | Pydantic output schema (TradeSetup) | [35] |
| `NI3singh/StockStalker-AI` | Listed in the sources only | [33] |
| `FareedKhan-dev/multi-agent-training-grpo` | Listed in the sources only | [34] |
| `Rajaykumar12/Finance_trader` | Listed in the sources only | [36] |
| `artvandelay/polymarket-agents` | Listed in the sources only | [37] |
| `SakanaAI/shachi` | Listed in the sources only | [38] |
| `ruvnet` prompt programming gist | Cited for Pydantic enforcement | [32] |

Note: the report cites [32] for the Pydantic schema statements. Entry 32 in the source list is
the `ruvnet` prompt-programming gist, not the `agentic-rag` or `crypto-signals` repositories named
in the sentence. The report also cites [37] for `bybit-py-mcp`, but entry 37 is
`polymarket-agents`. These cross references are kept as written. Treat them as approximate.
