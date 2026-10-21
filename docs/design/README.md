# Design Documents

These documents turn the design conversation (`Khởi động dự án Order Flow.pdf`) into reference
pages. The conversation was held with an AI assistant. Each page below keeps what was concluded
and drops the back-and-forth. For the reasoning path, read [design-history.md](design-history.md).

| Document | Read it to learn |
|---|---|
| [design-history.md](design-history.md) | How the design changed from first idea to the final blueprint, step by step |
| [tech-stack.md](tech-stack.md) | The chosen stack per service, and why Go and Python split the work |
| [repositories.md](repositories.md) | The three repositories, their roles and folder layout |
| [communication.md](communication.md) | Which protocol connects which parts, and why |
| [data-contracts.md](data-contracts.md) | The JSON and message shapes: state vector, AI output, OrderFlowBar, envelope |
| [quant-engine-draft.md](quant-engine-draft.md) | The first Python code for layers 1 to 3, with its known defects |

The research behind these choices is in [../research/README.md](../research/README.md). Rules that
override the original conversation are in `.context/decisions/` (D1 to D5).
