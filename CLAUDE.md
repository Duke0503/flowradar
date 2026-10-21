# FlowRadar

An order flow analytics platform with an AI Opportunity Radar. Start with `docs/index.md`.

## Rules

- **Everything is written in English**: code, comments, docs, commit messages, the domain model,
  decisions and generated pages. The two source files in the project root are Vietnamese and
  English mixed; they are input, not a style guide.
- The domain model lives in `docs/domain-model.json` and is managed with the `bmad-domain-driven`
  skill in `.claude/skills/`. Generated views in `docs/domain-viz/` are never edited by hand.
- Approved decisions live in `.context/decisions/`. A suggestion is not a decision until the
  project owner accepts it.
- FlowRadar publishes signals only and never places orders (D1 in `docs/domain-model.json`).
- Timestamps are UTC Unix milliseconds everywhere except the browser.
