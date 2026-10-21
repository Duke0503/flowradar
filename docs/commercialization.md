# Commercialization Plan

FlowRadar is planned for commercial release in **Vietnam and internationally** (D6). The team
plans first and builds later. This page lists what must be known before the first paying user.

Facts below come from public web pages read on 2026-10-21, mostly law-firm notes and trade press,
not official gazette texts. Each item is marked **verified** (read at a primary source), **reported**
(from a secondary source) or **unknown**. This page is not legal advice. Ask a lawyer in each target
market before launch.

## 1. Code licenses

See [licensing.md](licensing.md). Summary: most reused projects are MIT or Apache-2.0. Three are
GPL-3.0, three have no license, and the heatmap library is proprietary. We learn patterns from the
last two groups and write our own code.

## 2. Data licenses

FlowRadar shows other parties' data to end users. Each source needs permission for that use.

| Source | What we found | Status |
|---|---|---|
| CoinGlass | The public terms of service bar copying, selling or redistributing data, and commercial use without authorization. The page is dated 2022 and is a general site policy, not an API license. It says nothing about showing data to end users | **Reported.** Ask CoinGlass in writing whether the Standard plan allows public display, and which plan does |
| Binance and Bybit WebSockets | Not checked | **Unknown.** Read each exchange's API terms for redistribution limits |
| DeepSeek API | Not checked | **Unknown.** Read the terms on commercial use of outputs and on data sent in prompts |

Until CoinGlass answers, treat the $299 per month plan in [product-vision.md](product-vision.md) as
a development cost, not a confirmed production license.

## 3. Regulation of trading signals

FlowRadar publishes general signals with entry, stop loss and targets (D1). It never places
orders and never holds user money. Whether that is regulated depends on the market.

### Vietnam

- The Law on Digital Technology Industry took effect on 1 January 2026. It defines digital assets
  and crypto assets and treats them as property under the Civil Code. **Reported.**
- Resolution 05/2025/NQ-CP (9 September 2025) opened a five-year pilot for a regulated crypto asset
  market. **Reported.**
- Licensing for crypto asset trading platforms opened on 20 January 2026. Applicants must be Vietnamese
  companies with at least 10 trillion VND of charter capital, and foreign ownership is capped at 49%.
  **Reported.** This applies to exchanges. FlowRadar is not an exchange, but check that a signal
  service is not treated as a related service.
- Crypto transactions are taxed under securities rules until dedicated rules exist. **Reported.**
  This matters for users, not for FlowRadar, unless FlowRadar sells to Vietnamese users and must
  issue invoices and pay VAT. Ask an accountant.
- Whether publishing signals about crypto assets needs a license: **unknown.** Securities advice in
  Vietnam is licensed. Crypto assets are separate from securities under the new law. A lawyer must
  decide where a signal service falls.

### European Union

- ESMA says MiCA does not define copy trading or auto trading, and each service is classified case
  by case. It applies the MiFID II guidance to crypto copy trading. **Reported.**
- A German analysis from 2020 says general signals sent to the public lack a personal reference to
  the investor, so they are generally not investment advice. A 2026 note adds that MiCAR licensing
  may apply if signals involve order transmission or execution. **Reported.**
- Reading: a signal service that publishes general signals and does not execute or transmit orders
  sits at the lighter end. This fits D1. Confirm per country.

### United States

- One CFTC consent order found a signal aggregator liable for failing to register as a commodity
  trading advisor. The usual view is that a provider who does not direct a customer's account is outside
  that registration. **Reported.** SEC treatment of crypto signal providers: **unknown.**

### What keeps FlowRadar on the light side

| Practice | Why |
|---|---|
| Signals only, no execution, no custody (D1) | Avoids order handling, custody and most licensing |
| General publication, no personal advice | Signals do not depend on one user's finances or goals |
| No user accounts in the MVP (D4) | Avoids personal data duties at the start |
| Clear disclaimer: not investment advice, past results do not predict future ones | Required by most regulators in some form |
| Honest performance numbers (D2) | Win rate and invalidation rate must not mislead. Misleading performance claims are a risk of their own |
| No "guaranteed" or "risk-free" wording in marketing | Common cause of regulator action |

If a later feature adds personal advice, copy trading, order routing or paid private channels,
reopen this page. Each of those moves FlowRadar toward a regulated service.

## 4. Company and operating questions

- Which legal entity sells the product, and in which country? This sets tax, licensing and
  contract law.
- Payments: a processor that accepts crypto-related businesses, in each market.
- Personal data: needed once users log in (D4 defers it). Vietnam has a personal data protection law
  and the EU has GDPR.
- Terms of service, privacy policy and the risk disclaimer.
- Trademarks: do not use "TradingView" or exchange names in a way that implies endorsement. Keep the
  attribution that Lightweight Charts requires.

## 5. Planning checklist before implementation

| # | Task | Owner | Blocks |
|---|---|---|---|
| 1 | Ask CoinGlass for written permission to display data publicly, and the plan that allows it | Product owner | Any public launch (OQ8) |
| 2 | Read Binance and Bybit API terms for redistribution | Architect | Public launch (OQ9) |
| 3 | Read DeepSeek terms for commercial output use | Architect | Public launch (OQ12) |
| 4 | Legal opinion on signals in Vietnam, the EU and the US | Product owner with a lawyer | Marketing and pricing (OQ10) |
| 5 | Decide the treatment of GPL and unlicensed projects | Architect | Writing code (OQ11) |
| 6 | Choose a heatmap approach: buy, build or defer | Architect | Chart scope |
| 7 | Draft the disclaimer and terms | Product owner with a lawyer | Public launch |
| 8 | Add a `THIRD_PARTY_NOTICES` process to each repository | Architect | First release |

Tasks 1 to 3 can be done now at no cost and may change the design. If CoinGlass refuses public
display, the derivatives data source changes, and so do the data layer and the cost model.

## Sources

- [Vietnam's crypto asset regulation framework: from gray zone to a controlled pilot market](https://conventuslaw.com/featured-content/vietnams-crypto-asset-regulation-framework-from-gray-zone-to-a-controlled-pilot-market/)
- [Vietnam introduces licensing regime for cryptocurrency exchanges](https://www.vietnam-briefing.com/news/vietnam-licensing-regime-cryptocurrency-exchanges-digital-economy.html/)
- [Vietnam launches five-year trial licensing for crypto asset exchanges (Tuoi Tre)](https://news.tuoitre.vn/vietnam-launches-5-year-trial-licensing-for-crypto-asset-exchanges-103260121141134193.htm)
- [CoinGlass Terms of Service](https://www.coinglass.com/terms)
- [Trading signals for crypto assets: do signal providers have to register with BaFin?](https://fin-law.de/en/2020/06/22/trading-signals-for-crypto-assets-do-signal-providers-have-to-register-with-bafin/)
- [EU update on MiCA implementation: implications for copy trading services](https://conventuslaw.com/report/eu-update-on-mica-implementation-implications-for-copy-trading-services/)
- [CTAs, trading signals and DeFi (Blank Rome)](https://blankrome.com/publications/ctas-trading-signals-and-defi-connecting-dots-or-unconnected-leaps-logic)
