# Cloud runbook — Kelly's Market Insights updates

This file is followed by the scheduled **cloud routines** (Claude Code in Anthropic's cloud, Linux, fresh checkout of this repo).
The routine prompt tells you the **slot**: `morning`, `afternoon` or `weekend`. Everything else is here.

Kelly Chan is a Malaysian **信托基金顾问 (unit trust consultant)**. Her public website
(https://kellychan84.github.io/your-wealth-compass/) shows all market news on ONE page, **市场洞察 Market Insights**
(`insights.html`). Since 2026-10-02 there are **no separate brief pages** — do not create anything in `briefs/`.
Each run adds one JSON file of **insight cards, one per news story**. Morning and weekend runs also put a **每日速览 snapshot**
first (it feeds the home-page hero card); **afternoon runs have NO snapshot** (Kelly, 2026-10-02).

---

## 0. Setup
- Today's date in Malaysia: `TZ=Asia/Kuala_Lumpur date +%F` → `DATE`. Use Malaysia time for everything.
- If `insights/data/$DATE-$SLOT.json` **already exists**, stop and report "already published" — do not overwrite.
- Read, in full, before writing anything:
  - `automation/brief-rules.md` — **only the source rules** (whitelist / blacklist / verified-links-only, 第一步) and the voice
    guidance apply. Its HTML layout sections are retired; ignore them and `automation/brief-template.html`.
  - `insights/README.md` — the exact card format and writing rules. **Non-negotiable.**
- For context and to avoid repeating yourself, read the newest 2–3 files in `insights/data/` (afternoon: today's morning file).

## 1. Research
- Use WebSearch / WebFetch against **whitelist sources only**; never use Bloomberg, Axios or NPR.
- Every link must be a URL you actually fetched or saw in search results **in this run**. Never write a URL from memory.
- If a category has no whitelisted update, **leave it out entirely** — never write "no update / 暂无更新 / could not verify"
  anywhere, and never fill a gap with invented or stale numbers (Kelly's rule, 2026-10-01). Only publish what you actually have.

**News cards need real news articles (Kelly, 2026-10-13).** Trading Economics *quote pages*
(`tradingeconomics.com/<country>/stock-market`, `/commodity/...`, `/government-bond-yield`, `/currency`) are price tickers,
not news: use them for the 每日速览 figures and to check numbers, but **a news card's `source` must be a news article** —
a dated story on CNBC, Yahoo Finance, Investing.com, FX168, SCMP, The Standard, Focus Taiwan, The Edge Malaysia, The Star,
Al Jazeera, CBS News, PBS,
Benzinga, TheStreet or Schwab
(a Trading Economics article under `/news/` also counts). At most **2 cards per update** may cite tradingeconomics.com, and
never two from the same Trading Economics page. `build_index.py` warns when this is broken.

How to find articles (do this before falling back to anything else):
- Always pass `allowed_domains` in WebSearch, e.g. `["finance.yahoo.com","investing.com","cnbc.com"]`, plus the date words
  ("today", the weekday, the date), e.g. "stock market today Tuesday October 13 2026".
- One search per topic: Wall Street close; Fed / Treasury yields; oil; gold; a big US company or earnings; Hong Kong stocks /
  Hang Seng (`thestandard.com.hk`, `scmp.com`); China economy and companies (`scmp.com`, `fx168news.com`, `investing.com`);
  Taiwan stocks, TSMC and Asian chips (`focustaiwan.tw`, plus Samsung / SK Hynix on `investing.com`, `cnbc.com`);
  Malaysia (Budget, Bank Negara, ringgit, Bursa / KLCI — `theedgemalaysia.com`, `thestar.com.my`).
- **Malaysia (Kelly, 2026-10-14 — her readers are Malaysian, so try hard for one Malaysia card every update):** WebFetch
  `https://www.thestar.com.my/business/marketwatch` and `https://theedgemalaysia.com/categories/malaysia` and open today's
  Bursa / KLCI, ringgit, Bank Negara or Budget story. Afternoon: Bursa closes at 17:00 MYT, so a midday or "at mid-afternoon"
  report is fine — say so in the card.
- SCMP may show only part of an article (subscription): use only the facts and numbers you can actually see.
- Open the article with WebFetch and check its date is today or yesterday. If a site is blocked, try the same story on
  another whitelisted site before giving up on it.
- **Taiwan / TSMC:** search often misses Focus Taiwan's same-day stories, so also WebFetch its business section
  `https://focustaiwan.tw/business` and open today's Taiwan-stocks / TSMC / chip article from that list.
- **Hong Kong / China close (afternoon):** the HK market closes at 16:10 MYT, so look for the **closing** report first — search
  "Hong Kong shares close" / "Hang Seng close" plus the date on `thestandard.com.hk` and `scmp.com`, and WebFetch
  `https://www.thestandard.com.hk/finance` for today's list. Use a midday ("by noon") report only if no closing report is out yet.
- **One article, one card (Kelly, 2026-10-13).** If two stories you want come from the same article (e.g. a wrap-up covering
  both chips and China car sales), search for a separate article on the second story (a different site or a dedicated piece)
  instead of dropping it. Drop it only if no separate dated article exists. Aim for the top of the card range, not the minimum.

Coverage (Kelly, 2026-10-01 — cover news broadly): the Fed and US macro; **US** stories (Wall Street indices, major US
companies/earnings, US data and policy); **China** (economy and data, PBoC / yuan, China and Hong Kong stocks, Chinese companies,
US–China trade/tech); **Asian tech & semiconductors** (TSMC / Taiwan, Samsung and SK Hynix / Korea, Japanese chip and equipment
makers, China chip makers, Malaysia's semiconductor and data-centre sector, chip-stock moves, export controls, supply chains);
**oil and gold**; **Malaysia / ASEAN** (KLCI, ringgit, Bank Negara / OPR, local policy). For Hong Kong, China and Taiwan news use
The Standard, SCMP, Focus Taiwan and FX168 (added for Kelly, 2026-10-13) rather than a Trading Economics price page.

Slot focus:
- **morning** (time `${DATE}T09:00:00+08:00`): overnight Wall Street, Fed/yields, oil/gold, Asia open, Malaysia, big company news.
- **afternoon** (time `${DATE}T17:00:00+08:00`): what changed since this morning — Asia close / Bursa Malaysia (KLCI, USD/MYR),
  Fed odds, oil, news and earnings since the morning. Skip stories already carded this morning unless there is a real update.
- **weekend** (time `${DATE}T09:00:00+08:00`): the week in review (the week's key moves and 2–3 story lines — read this week's
  files in `insights/data/`), weekend news, and the week ahead (data, Fed events, earnings).

Voice: warm and professional, like explaining to a friend. Always say **信托基金顾问** and **信托基金** (never 理财顾问 / 单位信托).

## 2. Write `insights/data/$DATE-$SLOT.json`
A valid JSON array (no comments), exactly as specified in `insights/README.md`:
1. **Morning and weekend only — first item: the 每日速览 snapshot** (`"kind": "summary"`, id `$DATE-m0` / `$DATE-w0`) —
   headline, 4 key figures that you verified this run (the home page shows the first 4), 2–3 bullet points, Kelly's view,
   and the sources. **Afternoon: no snapshot** — the file holds only news cards.
2. **Then one card per news story** — morning and weekend **4–8 cards**, afternoon **3–6 cards**; ids `$DATE-m1…` / `$DATE-a1…` /
   `$DATE-w1…`. Every card has **all four** 🎯 boxes (`dda`, `prs`, `lump`, `bond`), each written for that specific story in
   both zh and en. Do not copy the same box text across cards.
- Do not add a `brief` field (there is no brief page any more).

## 3. Build and check
```bash
python3 tools/build_index.py
```
It must print `insights.js rebuilt with N insight cards` (N went up) and **no WARNING lines**. Every WARNING names the card and
the problem (missing box, label repeated in a box, banned word, "no update" wording, placeholder text…). Fix the JSON and rerun until it is clean.

**Then read back what you wrote (Kelly, 2026-10-21).** On 2026-10-21 a generator script published every analysis and box as the
literal text "zh" / "en". Before committing, print the first news card in full from the JSON file and check that every field
holds the real sentences you wrote. If you generate the file with a script, pass the actual text, never placeholder strings.

## 4. Publish
Only `insights/` may change. If anything else shows in `git status`, do not commit it.
```bash
git add insights
git commit -m "Add $DATE $SLOT insights"
git push origin HEAD:main
```
GitHub Pages redeploys about a minute after the push to `main`.
If the push to `main` is rejected, push to a new branch instead (`git push origin HEAD:claude/insights-$DATE-$SLOT`) and say
clearly in the summary that it needs merging into `main` before it appears on the site.

## 5. Summary (final message)
- The snapshot headline and the card titles
- Number of cards added
- Push result: "live on the site" / "pushed to branch X, needs merge" / error details
- Categories left out because there was no whitelisted update (mention them here only, never on the site)
