# Licensing of Reused Open-Source Code

FlowRadar is planned for commercial release (D6). The license of every project we learn from
decides whether we may copy its code or only learn its pattern.

Licenses below were read from the GitHub API and each repository's file list on 2026-10-21.
Re-check before any release. This page is not legal advice.

## Rules

1. **Permissive (MIT, Apache-2.0):** may copy. Keep the copyright notice. Apache-2.0 also needs
   the NOTICE file and a statement of changes.
2. **MPL-2.0:** may use. A file of theirs that we change must stay open under MPL-2.0.
3. **GPL-3.0:** do not copy code into a closed product. Learn the idea and write our own code.
   Running it as a separate program is a different case and needs a lawyer.
4. **No license file:** the author keeps all rights. Do not copy. Learn the idea, or ask the
   author for written permission.
5. **Commercial license:** pay for it, or do not use it.

An idea is not protected by copyright. A formula such as the two-pointer value area, or a design such
as "batch ticks in a Web Worker", is free to reimplement. Only the author's code is protected.

## Status per project

| Project | License | Use in FlowRadar | Action |
|---|---|---|---|
| `tradingview/lightweight-charts` | Apache-2.0 | Chart core | Use. Keep the attribution and NOTICE that TradingView requires |
| `awahab07/lightweight-orderflow-charts` | MIT | Footprint, TPO, CVD plugin | Use. Keep the copyright line |
| `edgedepthhq/edgedepth-gateway` | MIT | Go collector pattern | Use or copy |
| `mahmoud20138/OrderFlow-Analysis-Pro` | MIT | Absorption, sweep, divergence rules | Use or copy |
| `ymys/Bot-Auto-Screening-Bybit-trading` | MIT | Pre-score and trap filter | Use or copy |
| `qrak/LLM_trader` | MIT | RAG and claim validation | Use or copy |
| `wickra-lib/wickra` | Apache-2.0 | Streaming indicators | Use. Keep NOTICE |
| `apache/superset` (websocket) | Apache-2.0 | Redis fan-out, envelope | Use. Keep NOTICE |
| `tardis-dev/tardis-node` | MPL-2.0 | Replay with the live format | Use. Keep changes to its files open |
| `Tucsky/aggr` | GPL-3.0 | Web Worker batching idea | **Pattern only.** Write our own |
| `flowsurface-rs/flowsurface` | GPL-3.0 | TPO math, Arrow IPC | **Pattern only.** Write our own |
| `anvilfilbert/Auto-Crypto-Tradingjournal` | GPL-3.0 | Drift guard, multi-TP | **Pattern only.** Write our own |
| `Tucsky/aggr-server` | None found | Hot buffer plus cold store | **Pattern only**, or ask for permission |
| `jose-donato/cryexc-backend` | None found | DuckDB in-memory CVD | **Pattern only**, or ask for permission |
| `lagarcess/crypto-signals` | None found | Lifecycle, bracket orders | **Pattern only**, or ask for permission |
| `tapedelta/kline-orderbook-chart` | Proprietary (MRD Technologies) | Liquidation heatmap | **Do not use** without buying a license. Plan our own heatmap or choose another library |

## Effect on the plan

- The liquidation heatmap has no free source. Add it as its own work item. Options: buy the MRD
  license, write our own canvas layer on Lightweight Charts, or ship without it first.
- The research and design pages say "take code from X". For the GPL and no-license rows read that
  as "learn the pattern from X". The pages name the pattern, and these rows decide how to build it.
- Every file we write from a pattern should say where the idea came from, without copying their text.
- Keep a `THIRD_PARTY_NOTICES` file in each repository that lists each copied dependency, its
  license and its notice text. Generate it in CI from the dependency manifests.
- Check npm, Go and Python dependencies the same way. A permissive project can pull in a
  copyleft dependency.

## Open points

OQ11 (how to treat GPL and unlicensed projects: clean-room rewrite or written permission).
See [commercialization.md](commercialization.md).
