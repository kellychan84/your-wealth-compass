# 市场洞察 Market Insights — how the feed is written

Since 2026-10-02 the **Market Insights page (`insights.html`) is the only place for news** — no separate brief pages.
Each morning / afternoon / weekend update adds ONE file: `insights/data/<YYYY-MM-DD>-<morning|afternoon|weekend>.json`
(a JSON array). Then run `python3 tools/build_index.py` (cloud) or `tools\build-index.ps1` (Windows) to rebuild
`insights/insights.js`. The Python build also checks every card and prints a WARNING for anything that breaks these rules.

The array holds:
1. **morning and weekend only:** one **每日速览 snapshot** (`"kind": "summary"`) as the first item — it also feeds the
   home-page hero card. **Afternoon files have no snapshot** (Kelly, 2026-10-02);
2. one **news card per story** — morning/weekend 4–8, afternoon 3–6.

## Source rules (non-negotiable)
- Whitelist sources only (see `automation/brief-rules.md` 第一步); never Bloomberg, Axios or NPR.
- Every number, fact and quote must come from a page you fetched or a search result you saw **in this run**.
- `source.url` / `sources[].url` must be that exact URL. Never invent, alter or recall URLs from memory.
- A **news card's source must be a dated news article** (CNBC, Yahoo Finance, Investing.com, FX168, SCMP, The Standard,
  Focus Taiwan, The Edge Malaysia, The Star, Al Jazeera, CBS, PBS,
  Benzinga, TheStreet, Schwab, or a Trading Economics `/news/` article). Trading Economics quote/price pages are for the
  每日速览 figures only — at most 2 news cards per update may cite tradingeconomics.com, and each card a different article
  (Kelly, 2026-10-13).
- **Never make a card about missing data** ("no update", "暂无更新", "could not verify"), and don't mention gaps inside a card
  (Kelly, 2026-10-01). If a region or topic has nothing verified, leave it out.

## The four 🎯 boxes (Kelly's decision, 2026-10-02)
Every news card has all four, in zh and en, written for **that** story (no copy-paste between cards):

| key    | heading on the site                    | what to write |
|--------|----------------------------------------|---------------|
| `dda`  | 📈 长期投资与DDA · Long-term investing & DDA | what this story means for long-term investors and regular (DDA) investing |
| `prs`  | 🌱 退休规划与PRS · Retirement & PRS       | retirement-planning angle; may mention the PRS tax relief of up to RM3,000 a year |
| `lump` | 💰 Lump Sum 单笔投资 · Lump sum            | principles only: phasing in to spread timing risk, checking concentration, matching goals and risk tolerance |
| `bond` | 🏦 债券基金 · Bond funds                  | how the news may affect **Malaysian bond funds** (MGS / corporate bonds and sukuk): interest rates and Bank Negara's OPR, inflation, ringgit and foreign flows, credit. If there is little direct effect, say so briefly and explain bonds' role as a portfolio balancer |

- Start each box straight with the content. **Do not repeat the heading** ("🎯 DDA定期定额:…") — the site already shows it.
- General education only: **no allocation percentages**, no "split into N tranches", no buy/sell calls on named funds or
  stocks, no promissory or absolute words (保证 / 稳赚 / 必涨 / 牢不可破 / 封死下行空间 / 无风险 / guaranteed / can't lose /
  unshakeable / risk-free). Use "may / usually / generally" (可能 / 通常 / 一般).

## Writing rules
- Analysis (💡): 2–4 sentences — what happened (with the verified numbers), why it matters; one everyday analogy is welcome.
- Bilingual: every text field has `zh` and `en`. Tone: Kelly's warm-professional voice; say 信托基金顾问 / 信托基金.

## Schema — news card
```json
{
  "id": "2026-10-02-m1",                    // <date>-<m|a|w><n>, unique, n from 1
  "time": "2026-10-02T09:00:00+08:00",      // morning/weekend 09:00, afternoon 17:00, Malaysia time
  "tags": ["fed", "us"],                    // 1–3 of: my, us, cn, tech, commodities, fed
  "title":    { "zh": "...", "en": "..." },
  "analysis": { "zh": "...", "en": "..." },
  "dda":      { "zh": "...", "en": "..." },
  "prs":      { "zh": "...", "en": "..." },
  "lump":     { "zh": "...", "en": "..." },
  "bond":     { "zh": "...", "en": "..." },
  "source":   { "name": "Investing.com", "url": "https://..." }
}
```

## Schema — 每日速览 snapshot (first item; morning and weekend files only)
```json
{
  "id": "2026-10-02-m0",                    // n = 0; m (morning) or w (weekend), never a (afternoon)
  "kind": "summary",
  "time": "2026-10-02T09:00:00+08:00",
  "tags": [],
  "title":  { "zh": "一句话总结今天", "en": "One-line summary of the day" },
  "figures": [                              // 4 verified numbers (home page shows the first 4); only ones you have
    { "label": { "zh": "标普500", "en": "S&P 500" }, "value": "7,666.45", "change": "+0.19%", "dir": "up" }
  ],                                        // dir: "up" | "dn" | "" (neutral); change may also be { "zh", "en" } for words
  "points": [ { "zh": "...", "en": "..." } ],   // 2–3 key headlines
  "takeaway": { "zh": "Kelly 的观点(1–3 句)", "en": "Kelly's view" },
  "sources": [ { "name": "Yahoo Finance", "url": "https://..." } ]
}
```
The snapshot shows at the top of its update in the 全部 view, under the 📰 每日速览 chip, and on the home-page hero card
(headline, first 4 figures, Kelly's view).

## Tags
`my` 🌏 马来西亚/东盟 · `us` 🗽 美国 · `cn` 🏮 中国 · `tech` 💻 科技与AI · `commodities` 🟡 黄金与原油 · `fed` 🏦 美联储与宏观.
(No `prs` tag — Kelly removed it 2026-10-01.)

- `us`: US stock market (Dow / S&P 500 / Nasdaq), US companies and earnings, US economy and policy (tariffs, Treasury, jobs,
  consumers). Fed decisions keep `fed`; add `us` too when the story is about US markets.
- `tech`: global tech/AI **including Asian tech & semiconductors** (TSMC, Samsung, SK Hynix, Japanese/Chinese chip makers,
  Malaysia's semiconductor and data-centre sector). Add `cn` or `my` too when the company/market is Chinese or Malaysian.
- `cn`: China and Hong Kong — Chinese economy and data, PBoC / yuan, China and HK stocks, Chinese companies, US–China trade and tech.
