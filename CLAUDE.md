# Stock Analyzer — AI Codebase Catalog

> This file is for AI agents starting a new session. It maps the entire project so you can navigate quickly without reading 2600+ lines.

## Project Overview

Single-file React SPA (`index.html`, ~2667 lines) for stock fundamental analysis. No build system — uses CDN React 18 + Babel in-browser transpilation + Chart.js. All state in localStorage. Dark theme UI.

## File Structure

```
index.html          — THE app (CSS + JS + HTML, all-in-one). This is the only file that matters.
app.js              — Legacy/unused JS copy (not loaded by index.html)
stock-analyzer-complete.html — Old snapshot, not in use
index.html.bak      — Backup of earlier version
part_a[a-e]         — Split artifacts from earlier work, not in use
journal.md          — Dev journal/notes
start.sh            — Simple HTTP server launcher
```

## index.html Internal Structure

### Lines 1–405: HTML + CSS

- **1–13**: HTML head, CDN imports (React 18, ReactDOM, Babel, Chart.js, Inter + JetBrains Mono fonts)
- **14–401**: `<style>` block — all CSS. Dark theme variables at `:root` (line 16–24). Key classes:
  - Layout: `.app`, `.container`, `.header`, `.main`
  - Components: `.btn*`, `.input*`, `.modal*`, `.search-*`, `.wl-item`, `.metric-card`, `.ai-panel`, `.ai-bullet`, `.chart-card`, `.pie-card`, `.alert-row`, `.checklist-*`
  - Responsive: media query at line 393 (768px breakpoint)
- **402–405**: `<body>`, `<div id="root">`, `<script type="text/babel">` opens

### Lines 406–465: SECTION 1–2 — Config & Local Storage

- **414–440**: Constants — API base URLs (`FMP_STABLE`, `CLAUDE_URL`, `OPENROUTER_URL`, `FINNHUB`), `CHECKLIST_ITEMS` array (defines all 10 financial metrics tracked), chart colors
- **445–465**: localStorage helpers — `getStore/setStore`, `getApiKeys/setApiKeys`, `getWatchlist/setWatchlist`, `getOverrides/setOverrides`, `getUserInputs/saveUserInputs`, `getStockCache/setStockCache`
- **Line 452**: **DEFAULT API KEYS** live here — `fmp`, `claude` (empty), `finnhub`, `openrouter` (empty)

### Lines 467–735: SECTION 3 — API Functions

#### FMP (Financial Modeling Prep) — primary data source
- **470** `fmpGet(path, key)` — base FMP fetch wrapper
- **476** `searchTickers(query, key)` — ticker search (symbol + name)
- **494** `fetchProfile(ticker, key)` — company profile
- **499** `fetchIncomeStatements(ticker, key)` — 15yr income statements
- **503** `fetchKeyMetrics(ticker, key)` — key financial metrics
- **507** `fetchEnterpriseValues(ticker, key)` — enterprise value data
- **511** `fetchCashFlowStatements(ticker, key)` — cash flow data
- **515** `fetchRatios(ticker, key)` — financial ratios
- **519** `fetchHistoricalPrices(ticker, key)` — 1yr daily prices
- **526** `fetchFmpNews(ticker, key)` — FMP news (50 items)
- **530** `fetchGeoRevenue(ticker, key)` — geographic revenue breakdown
- **535** `fetchSegRevenue(ticker, key)` — business segment revenue

#### Finnhub — analyst data + news
- **540** `fetchPriceTargetConsensus(ticker, finnhubKey)` — analyst price targets
- **551** `fetchRatingConsensus(ticker, finnhubKey)` — analyst ratings (buy/hold/sell)
- **566** `fetchFinnhubNews(ticker, dateStr, finnhubKey)` — date-specific company news

#### Aggregator
- **573** `fetchAllStockData(ticker, keys)` — calls all FMP + Finnhub APIs in parallel, returns unified data object

#### AI API calls
- **602** `callClaude(prompt, claudeKey, maxTokens)` — direct Anthropic API call (claude-sonnet-4-20250514)
- **626** `callOpenRouter(prompt, openrouterKey, maxTokens)` — OpenRouter API call (anthropic/claude-3.5-haiku)
- **650** `callAI(prompt, apiKeys, maxTokens)` — router that picks Claude or OpenRouter based on `apiKeys.aiProvider`
- **658** `hasAiKey(apiKeys)` — checks if user has any AI key configured

#### AI Feature Prompts
- **664** `fetchAiInsights(stockInfo, apiKeys)` — returns `{oneliner, segments[], growth}`. **Growth prompt** (line 676): 3 sentences — analyst names + rates + average, reasoning, downside risks
- **689** `fetchAiProsCons(stockInfo, apiKeys)` — returns `{pros[3], cons[3]}`. **Pros/Cons prompt** (line 694): items 1-2 = company fundamentals, item 3 = macro-economic factors
- **709** `fetchAiStockAnswer(question, stockInfo, apiKeys)` — free-form Q&A about a stock
- **722** `fetchAiNewsSummary(newsItems, apiKeys)` — summarizes news articles for price alert explanations

### Lines 736–955: SECTION 4 — Calculation & Utility Functions

- **739–800**: Number formatting (`fmtNum`, `fmtBigNum`, `fmtPct`), PE calculations (`safePE`, `calcPE`, `calcForwardPE`), market cap calc, daily price change detection (`calcDailyChanges`, `findSignificantChanges`)
- **801** `calcMktCapToEbitda(evData, incomeData)` — Market Cap / EBITDA time series
- **823** `buildYearSeries(data, field)` — extracts yearly time series from FMP data
- **832** `parseDistribution(raw)` — parses geographic/segment revenue for pie charts
- **845** `getExpectedYears(rawData)` — determines which years should have data (recent 5)
- **878** `buildHistoryStatus(series, yearList)` — checks data completeness per year
- **893** `buildChecklist(rawData)` — **CORE FUNCTION** — builds the 10-item data quality checklist from raw API data. Each item: `{key, label, available, total, data, detail, series, analyst}`. Items: Revenue, Net Income, EPS, PE Ratio, ROE, Debt/Equity, FCF, Dividend Yield, Geo Distribution, Seg Distribution, Analyst Rating

### Lines 956–1104: SECTIONS 5–6 — Base Components & API Key Modal

- **961** `Spinner` — loading spinner
- **966** `LoadingOverlay` — full-screen loading overlay
- **992** `ChartWrap` — Chart.js line/bar chart wrapper (auto-destroys on unmount)
- **1014** `PieWrap` — Chart.js pie/doughnut chart wrapper
- **1038** `ApiKeyModal` — modal for entering FMP, Claude/OpenRouter, Finnhub keys. Shows current key values below inputs. AI provider toggle (Claude vs OpenRouter)

### Lines 1110–1530: SECTIONS 7–8 — Search & Data Checklist

- **1113** `SmartSearch` — debounced ticker search with dropdown results. Uses `searchTickers()`. 500ms debounce.
- **1221** `DataChecklistModal` — **LARGE COMPONENT** (~310 lines). Shows all 10 checklist items with status (OK/missing/partial). For items with missing data, shows year-by-year input fields so user can manually fill gaps. Handles distribution items (geo/segment) with add/remove rows. Auto-saves to localStorage. Has "Continue with available data" and "Save & View Analysis" buttons.

### Lines 1532–1650: SECTION 9 — Metrics Table

- **1535** `MetricsTable` — 2-column grid of financial metrics (4×N). Each metric card shows label + value + optional override input. Fields: Stock Price, EPS, PE Ratio, Forward PE, Market Cap, Revenue, Net Income, FCF, ROE, Debt/Equity, Dividend Yield, Shares Outstanding, Earnings Growth Rate, Analyst Rating, Price Target. Editable via dashed-border inputs. Shows Finnhub analyst data inline.

### Lines 1653–1876: SECTION 10 — AI Insights Panel

- **1656** `AiKeyPrompt` — inline prompt to enter AI key if missing. Validates key by test API call. Supports Claude + OpenRouter provider selection.
- **1715** `AiInsightsPanel` — main AI display panel. Shows:
  - Company one-liner description (bold intro)
  - Business segment tags (blue pills)
  - Analyst Growth Expectations section (3-sentence structured analysis)
  - Pros & Cons (two-column grid: left=pros, right=cons, each with green/red icons)
  - Q&A chat box (ask questions about the stock, AI responds)

### Lines 1878–2100: SECTIONS 11–13 — Charts & Price Alerts

- **1881** `HistoryChart` — reusable line chart for historical data (revenue, net income, EPS, etc.)
- **1930** `PriceAlertChart` — 1-year price chart with markers on significant movement days (>5% change)
- **1994** `PriceAlertList` — list of significant price movement days. For each: date, % change, AI-generated news summary explaining the move. Fetches Finnhub news by date, then calls AI to summarize.
- **2054** `FinnhubKeyPrompt` — inline prompt to add Finnhub key (shown when missing)

### Lines 2104–2196: SECTIONS 14–16 — Distribution & Supplements

- **2107** `DistributionSection` — two pie charts side by side: Geographic Revenue + Business Segment Revenue
- **2147** `MissingDataFooter` — yellow warning box listing any checklist items with incomplete data
- **2173** `SupplementSection` — collapsible section with Market Cap/EBITDA historical chart

### Lines 2198–2256: SECTION 17 — Dashboard Page

- **2201** `DashboardPage` — home page. Contains:
  - Header with "API Keys" button and "Pro Mode" toggle
  - `SmartSearch` component
  - Watchlist (saved stocks). Each item shows ticker + name + delete button. Click to load stock.

### Lines 2258–2523: SECTION 18 — Detail Page

- **2261** `DetailPage` — **LARGEST COMPONENT** (~260 lines). The full stock analysis view. Manages state for:
  - `overrides` — user metric overrides (editable fields)
  - `localRating` — Finnhub analyst rating (fetched on mount)
  - `insights` — AI insights (fetched on mount if AI key exists)
  - `prosCons` — AI pros/cons (fetched on user click)
  - `qaMessages` — Q&A chat history
  - `significantDays` + `alertSummaries` — price alert data + AI news summaries

  Renders in order:
  1. Back button + stock header (name, ticker, sector, exchange)
  2. `MetricsTable` (editable financial metrics grid)
  3. `AiInsightsPanel` (AI analysis + pros/cons + Q&A)
  4. Historical charts grid (Revenue, Net Income, EPS, ROE, D/E, Div Yield, FCF, PE)
  5. `PriceAlertChart` + `PriceAlertList` (significant price movements)
  6. `DistributionSection` (geo + segment pie charts)
  7. `SupplementSection` (Market Cap/EBITDA)
  8. `MissingDataFooter`

### Lines 2525–2667: SECTIONS 19–20 — App Root & Render

- **2528** `App` — root component. Manages:
  - `apiKeys` state (synced with localStorage)
  - `watchlist` state
  - `page` navigation ('dashboard' | 'detail')
  - `selectedStock`, `rawData`, `checklist`, `checklistInputs`
  - `proMode` toggle (shows data checklist before detail page)
  - Stock selection flow: search → fetch all data → build checklist → (pro mode: show checklist modal) → detail page
- **2662** `ReactDOM.createRoot(document.getElementById('root')).render(<App />)` — mounts the app

## Data Flow

```
User searches ticker
  → searchTickers() via FMP API
  → User clicks result
  → fetchAllStockData() (parallel FMP + Finnhub calls)
  → buildChecklist() (validates data completeness)
  → [Pro Mode: DataChecklistModal for manual gap-fill]
  → DetailPage renders with:
      - MetricsTable (from raw data + user overrides)
      - AiInsightsPanel (from AI API calls)
      - Historical charts (from yearly series data)
      - Price alerts (from daily price changes + AI news summaries)
      - Distribution pies (from geo/segment revenue data)
```

## Key State in localStorage

| Key | Content |
|-----|---------|
| `sa_api_keys` | `{fmp, claude, finnhub, openrouter, aiProvider}` |
| `sa_watchlist` | `[{ticker, name}]` |
| `sa_ov_<TICKER>` | Metric overrides for a specific stock |
| `sa_inputs_<TICKER>` | User-filled data from checklist modal |
| `sa_cache_<TICKER>` | Cached raw API data |

## External APIs

| API | Base URL | Auth | Used For |
|-----|----------|------|----------|
| FMP (Financial Modeling Prep) | `https://financialmodelingprep.com/stable` | `apikey=` query param | All financial data, profiles, news, prices |
| Finnhub | `https://finnhub.io/api/v1` | `token=` query param | Analyst ratings, price targets, date-specific news |
| Anthropic Claude | `https://api.anthropic.com/v1/messages` | `x-api-key` header | AI insights (model: claude-sonnet-4-20250514) |
| OpenRouter | `https://openrouter.ai/api/v1/chat/completions` | `Authorization: Bearer` header | AI insights (model: anthropic/claude-3.5-haiku) |
