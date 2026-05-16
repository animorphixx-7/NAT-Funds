<div align="center">

<br/>

<pre>
 ███╗   ██╗ █████╗ ████████╗    ███████╗██╗   ██╗███╗   ██╗██████╗ ███████╗
 ████╗  ██║██╔══██╗╚══██╔══╝    ██╔════╝██║   ██║████╗  ██║██╔══██╗██╔════╝
 ██╔██╗ ██║███████║   ██║       █████╗  ██║   ██║██╔██╗ ██║██║  ██║███████╗
 ██║╚██╗██║██╔══██║   ██║       ██╔══╝  ██║   ██║██║╚██╗██║██║  ██║╚════██║
 ██║ ╚████║██║  ██║   ██║       ██║     ╚██████╔╝██║ ╚████║██████╔╝███████║
 ╚═╝  ╚═══╝╚═╝  ╚═╝   ╚═╝       ╚═╝      ╚═════╝ ╚═╝  ╚═══╝╚═════╝ ╚══════╝
</pre>

### Institutional-grade mutual fund analytics.

<br/>

[![Node.js](https://img.shields.io/badge/Node.js_18+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com)
[![JavaScript](https://img.shields.io/badge/Vanilla_JS-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)](https://www.chartjs.org)
[![Pino](https://img.shields.io/badge/Pino_Logger-68A063?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://getpino.io)
[![ExcelJS](https://img.shields.io/badge/ExcelJS-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)](https://github.com/exceljs/exceljs)
[![Axios](https://img.shields.io/badge/Axios-5A29E4?style=for-the-badge&logo=axios&logoColor=white)](https://axios-http.com)
[![Nodemon](https://img.shields.io/badge/Nodemon-76D04B?style=for-the-badge&logo=nodemon&logoColor=white)](https://nodemon.io)
[![AMFI](https://img.shields.io/badge/AMFI_India-FF6600?style=for-the-badge&logoColor=white)](https://www.amfiindia.com)
[![NSE](https://img.shields.io/badge/NSE_TRI-1D3557?style=for-the-badge&logoColor=white)](https://www.nseindia.com)
[![BSE](https://img.shields.io/badge/BSE_Indices-003087?style=for-the-badge&logoColor=white)](https://www.bseindia.com)
[![RBI](https://img.shields.io/badge/RBI_T--Bill-0E4D92?style=for-the-badge&logoColor=white)](https://www.rbi.org.in)

<br/>

> 9,100+ live schemes · 15 financial metrics · 6 live data sources · zero build step  
> Sharpe · Sortino · Beta · Jensen's Alpha · Max Drawdown · Calmar · Rolling Returns · Capture Ratios

<br/>

---

</div>

## The problem

Most mutual fund platforms in India show you a star rating and a 1-year return. That's not analysis — that's a marketing number.

The metrics that actually matter when evaluating a fund — rolling returns across multiple windows, downside capture, Jensen's Alpha, Calmar ratio, Sortino ratio — are either buried in PDFs, hidden behind expensive platforms, or just not available to retail investors.

NAT Funds fixes that. It pulls live data from AMFI, NSE/BSE, and RBI, computes everything from raw NAV history, and serves it through a no-install browser UI. The entire backend — data pipelines, metric calculations, caching, API — is built from scratch in Node.js, without a framework doing the heavy lifting.

---

## What it computes

Everything in `services/metricsCalculator.js` is a pure function — no side effects, no hidden state. All inputs are NAV history arrays and TRI series. All outputs are numbers.

### Returns
| Metric | How |
|---|---|
| **CAGR (1Y / 3Y / 5Y / Inception)** | Closest-previous NAV lookup with a 7-day gap tolerance. Returns N.A. if history is too short — no fake numbers. |
| **Rolling Returns 1Y** | Daily-stepped windows across full NAV history. Reports min, max, and distribution. |
| **Rolling Returns 3Y** | Monthly-stepped (one window per calendar month). Reports p10 / p25 / p75 / p90 percentile bands. |

### Risk
| Metric | How |
|---|---|
| **Standard Deviation** | Annualised sample StdDev of monthly returns. 36-month lookback, minimum 12 months. |
| **Max Drawdown** | Two-pass O(n) scan — peak, trough, and recovery dates all tracked. Not just the number; the full timeline. |
| **SEBI Riskometer** | AMFI-published label when available, percentile rank of StdDev within sub-category otherwise. |

### Risk-adjusted
| Metric | How |
|---|---|
| **Sharpe Ratio** | `(avg_excess_return / σ_fund) × √12`. Excess return computed using geometric monthly RFR from the live RBI 91-day T-bill rate. |
| **Sortino Ratio** | Same as Sharpe but denominator is downside deviation only — doesn't penalise upside volatility. |
| **Calmar Ratio** | `3Y CAGR% / |Max Drawdown%|`. Returns `null` if 3Y CAGR is unavailable or drawdown is zero. |

### Benchmark-relative
| Metric | How |
|---|---|
| **Beta** | Covariance / variance of 36 aligned monthly returns (fund vs real benchmark TRI). |
| **Jensen's Alpha** | `AnnFundReturn − [Rf + β × (AnnBenchReturn − Rf)]`. Geometric annualisation throughout. |
| **Information Ratio** | `(AnnFundReturn − AnnBenchReturn) / TrackingError`. Requires ≥36 aligned months. |
| **R²** | Pearson correlation² between fund and benchmark monthly returns. 0–100 scale. |
| **Upside / Downside Capture** | Morningstar methodology. Requires ≥10 up-months and ≥6 down-months to be meaningful. |

### Peer scoring
| Metric | How |
|---|---|
| **Consistency Score (0–10)** | Composite: 30% median 3Y rolling · 20% rolling positive % · 20% downside capture (inverted) · 20% Sortino · 10% 3Y CAGR. Percentile-normalised within sub-category. |

### AMFI-published (sourced directly, not calculated)
TER, AUM (₹ Cr), AMFI Information Ratios (1Y/3Y/5Y/10Y for Direct and Regular plans), SEBI Riskometer label, and fund-to-benchmark mappings are all pulled from official AMFI APIs and XLSX reports — not scraped, not approximated.

---

## Architecture

```
server.js                         ← Express entry point: security, rate limiting, route mounting, boot trigger
│
├── shared/
│   ├── appState.js               ← Single source of truth for all in-memory state
│   └── logger.js                 ← Pino structured logger (level controlled via LOG_LEVEL)
│
├── boot/
│   ├── startup.js                ← Full boot() sequence: AMFI parse → NAV fetch → metrics compute
│   └── dataHelpers.js            ← Per-fund enrichment: attaches TER, AUM, AMFI IR, Benchmark, Riskometer
│
├── routes/
│   ├── api.js                    ← /api/* public REST endpoints
│   ├── admin.js                  ← /admin/* sync endpoints (auth-gated)
│   └── ter.js                    ← /ter/:schemeCode single-scheme TER lookup
│
├── services/
│   ├── amfiParser.js             ← Parses live AMFI NAV text feed; selects top schemes
│   ├── dataFetcher.js            ← NAV history fetcher (mfapi.in) with per-scheme disk cache
│   ├── metricsCalculator.js      ← ~1,200 lines of pure quantitative finance functions
│   ├── fundPerformanceService.js ← AUM, Benchmark, AMFI IR, SEBI Riskometer via AMFI Fund Performance API
│   ├── terService.js             ← Downloads and parses AMFI TER XLSX monthly (via ExcelJS)
│   ├── triService.js             ← Benchmark TRI history from NSE and BSE Indices APIs
│   └── riskFreeRate.js           ← Live 91-day T-bill YTM from RBI/CCIL — cached 24h in memory
│
├── data/                         ← Committed JSON stores (refreshed by cron + admin endpoints)
│   ├── ter-data.json             ← TER snapshot
│   ├── aum-data.json             ← AUM per scheme (daily, ₹ Crores)
│   ├── benchmark-data.json       ← Fund → benchmark map (90-day TTL — benchmarks are SEBI-stable)
│   ├── ir-data.json              ← AMFI Information Ratios: 1Y/3Y/5Y/10Y, Direct + Regular
│   ├── riskometer-data.json      ← SEBI-mandated riskometer labels
│   └── tri-data.json             ← Full benchmark TRI history
│
├── cache/                        ← Git-ignored, auto-populated at runtime
│   ├── nav_<schemeCode>.json     ← Per-scheme NAV history (24h TTL)
│   ├── processed_funds.json      ← Full computed fund list (24h TTL — warm restarts skip recompute)
│   └── ter-parsed.json           ← Parsed TER (24h TTL — avoids re-downloading AMFI XLSX)
│
├── tests/
│   └── smoke.test.js             ← 9 shape/import tests using node:test (no network, ~0.5s)
│
├── dev-tools/                    ← 19 standalone diagnostic scripts (never imported by server)
│   ├── test-api.js               ← API smoke test
│   ├── test-metrics.js           ← Metrics unit checks
│   ├── diagnose.js               ← Data-coverage diagnostic report
│   ├── test-benchmark.js         ← TRI benchmark lookup verification
│   └── app_original.js           ← Original monolithic frontend (~88 KB — kept as reference)
│
├── scripts/                      ← Data-quality audit tools (read-only, never imported)
│   ├── audit_aum_norm.js         ← AUM match-rate audit across all 9,100+ funds
│   └── test_normalise.js         ← Unit tests for normaliseName() + false-positive check
│
└── public/                       ← SPA frontend (no build step)
    ├── index.html                ← Shell — loads JS modules in strict dependency order
    ├── styles.css
    └── js/
        ├── constants.js          ← CATEGORY_META, METRIC_TOOLTIPS
        ├── state.js              ← Frontend state object
        ├── api.js                ← fetch() wrapper around /api/*
        ├── formatters.js         ← fmt, fmtNav, fmtAUM, fmtScore, etc.
        ├── router.js             ← Hash-based routing
        ├── ui.js                 ← showView, setPlanType, renderSidebar
        ├── search.js             ← Live search bar
        ├── compare.js            ← Compare-list state management
        ├── charts.js             ← Chart.js wrappers
        ├── init.js               ← Bootstrap: progress polling → handleRoute()
        └── views/
            ├── home.js           ← Category dashboard
            ├── explore.js        ← Fund table with filters and pagination
            ├── fund-detail.js    ← Full per-fund analytics page
            └── compare.js        ← Side-by-side comparison view (up to 4 funds)
```

---

## Data sources

| Source | What it provides | When it refreshes |
|---|---|---|
| **AMFI NAV text feed** | Live NAV for ~9,100 schemes | Boot + on demand |
| **mfapi.in** | Full historical NAV per scheme | 24h disk cache per scheme |
| **AMFI Fund Performance API** | AUM, fund→benchmark map, AMFI IR (1Y/3Y/5Y/10Y), SEBI Riskometer | Daily cron at 09:30 IST + `/admin/sync-aum` |
| **AMFI TER XLSX** | Total Expense Ratio per scheme | Daily cron at 09:00 IST + `/admin/sync-ter` |
| **NSE / BSE Indices API** | Benchmark TRI history (Beta, Alpha, IR, Capture) | Daily cron at 09:31 IST + `/admin/sync-tri` |
| **RBI / CCIL (91-day T-bill)** | Risk-free rate for Sharpe, Sortino, Jensen's Alpha | 24h in-memory cache |

### Caching — three tiers

The system uses a deliberate three-tier cache to keep restarts fast without burning API rate limits:

**Tier 1 — `data/*.json` (committed, long-lived):** TER, AUM, TRI, benchmarks, AMFI IR, and riskometers. Refreshed by scheduled cron jobs and admin endpoints. Benchmarks get a 90-day TTL specifically — they're semi-permanent under SEBI mandate and don't need daily re-fetching.

**Tier 2 — `cache/nav_<schemeCode>.json` (git-ignored, 24h TTL):** Per-scheme NAV history from mfapi.in. Auto-populated on first fetch, reused for 24 hours. On a fresh clone this folder is empty — it fills itself.

**Tier 3 — `cache/processed_funds.json` (git-ignored, 24h TTL):** The full computed fund list — every scheme with every metric applied. If this file is less than 24 hours old, the entire boot sequence (parse → fetch → compute) is skipped. A warm restart takes seconds instead of minutes.

---

## Boot sequence

What actually happens when you run `npm start`:

```
1. Load TER index from data/ter-data.json (sync from AMFI if missing)
2. Load AUM, Benchmark, AMFI IR, Riskometer from data/ stores
3. Fetch live RBI 91-day T-bill rate
4. Check cache/processed_funds.json age
   ├── < 24h old → load and skip to step 8 (warm restart)
   └── stale or missing → continue
5. Parse live AMFI NAV feed (~9,100 schemes)
6. Batch-fetch historical NAV for top ~3,000 schemes (10 concurrent)
7. Compute all metrics in memory → save processed_funds.json
8. Set dataReady = true → begin serving
```

During steps 5–7, the frontend shows a live progress bar. It polls `/api/status` every second. No blank screen, no mystery spinner.

---

## API reference

### Public — `/api/*`

| Method | Path | What it does |
|---|---|---|
| `GET` | `/api/status` | Readiness check + boot progress percentage |
| `GET` | `/api/categories` | Category and sub-category counts (supports `?planType=` and `?optionType=`) |
| `GET` | `/api/funds` | Paginated fund list — filterable, sortable |
| `GET` | `/api/fund/:schemeCode` | Full fund detail with live NAV sync and fresh metric recalculation |
| `GET` | `/api/fund/:schemeCode/nav-history` | Monthly-sampled NAV for charts (`?period=1y\|3y\|5y\|max`) |
| `GET` | `/api/compare?codes=...` | Side-by-side data for 2–4 scheme codes |
| `GET` | `/api/search?q=...` | Full-text search across scheme name and AMC (top 20) |
| `GET` | `/ter/:schemeCode` | TER lookup for a single scheme |

Query parameters for `/api/funds`:

| Param | Example | Notes |
|---|---|---|
| `type` | `Equity` | Fund type filter |
| `subCategory` | `Large Cap` | Comma-separated for multiple |
| `planType` | `Direct` | `Direct` or `Regular` |
| `optionType` | `Growth` | `Growth` or `IDCW` |
| `search` | `HDFC` | Full-text across scheme name and AMC |
| `sortBy` | `cagr3y` | Supports nested fields e.g. rolling return percentiles |
| `order` | `desc` | `asc` or `desc` |
| `page` | `1` | 1-indexed |
| `limit` | `20` | Max 100 per page |

### Admin — `/admin/*`

Protected by `X-Admin-Secret` header when `ADMIN_SECRET` is set.

```
GET /admin/sync-ter   — Re-download AMFI TER XLSX → rebuild ter-data.json
GET /admin/sync-aum   — Refresh AUM + AMFI IR + SEBI Riskometer
GET /admin/sync-tri   — Refresh all benchmark TRI histories from NSE/BSE
```

---

## Running it

### Prerequisites
- Node.js 18+
- Internet access on first boot (AMFI data fetch)

### Install and start

```bash
git clone https://github.com/harleenkhanuja/NAT-Funds.git
cd NAT-Funds
npm install

npm start       # production: node server.js
npm run dev     # development: nodemon with auto-restart
npm test        # 9 smoke tests, no network, ~0.5s
```

Open `http://localhost:3001` — the server serves the frontend automatically.

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3001` | HTTP port |
| `ADMIN_SECRET` | _(empty)_ | If set, `/admin/*` requires `X-Admin-Secret` header |
| `CORS_ORIGIN` | _(empty)_ | Explicit CORS origin if separating frontend from backend |
| `LOG_LEVEL` | `info` | Pino level: `trace` `debug` `info` `warn` `error` `fatal` |

---

## Dev tools and audit scripts

### `dev-tools/` — diagnostics (19 files, never imported by the server)

```bash
node dev-tools/test-api.js        # hit the running API, check shape of responses
node dev-tools/test-metrics.js    # verify metric calculations against known inputs
node dev-tools/diagnose.js        # data-coverage report: how many funds have TER / AUM / TRI
node dev-tools/test-benchmark.js  # confirm benchmark TRI lookup works for a sample fund
```

### `scripts/` — data quality (read-only, run after a data refresh)

```bash
node scripts/audit_aum_norm.js    # AUM name-match audit across all 9,100+ schemes
node scripts/test_normalise.js    # unit tests for normaliseName() — checks false positives too
```

---

## Design decisions worth explaining

**No frontend framework, no build step.** The SPA runs on plain HTML and ES modules loaded in strict dependency order. No Webpack, no Vite, no transpilation. This was deliberate — it removes an entire category of complexity, keeps the frontend debuggable in DevTools without source maps, and means a fresh clone just works.

**`metricsCalculator.js` is entirely pure functions.** ~1,200 lines, zero side effects. Every metric takes a NAV array and returns a number. This makes testing straightforward and makes it impossible for a metric computation to accidentally corrupt global state.

**The 90-day benchmark TTL.** AMFI publishes benchmark assignments per SEBI mandate and they almost never change. Hitting the API daily for something that changes quarterly wastes requests and introduces latency on sync operations. The 90-day TTL is a deliberate tradeoff.

**7-day gap tolerance on CAGR.** Indian markets have public holidays that create NAV gaps. A strict date lookup would produce N.A. for too many valid funds. The 7-day window finds the closest available NAV, which matches how AMFI itself calculates published returns.

**Warm restart design.** `cache/processed_funds.json` exists specifically so the server can restart during the day without re-running a 2–4 minute boot sequence. This is a real operational concern — if you're deploying on a small VPS and need to push a config change, you don't want users staring at a loading screen.

**Rate limiting at 200 req/min/IP.** The server aggregates from six external APIs. Exposing an uncapped public API that could trigger cascading upstream rate limit errors is a bad idea. The Express-layer limit is a backstop.

> **IDCW note:** Rolling returns for IDCW plans reflect NAV movement only — dividends paid out are excluded. This matches the methodology used by Tickertape and INDmoney.

---

## What I'd add next

- [ ] **SIP return calculator** — XIRR-based, takes a monthly investment amount and computes actual investor returns
- [ ] **Portfolio overlap tool** — show what percentage of holdings two funds share
- [ ] **Alert system** — notify when a fund's rolling return drops below a user-set threshold
- [ ] **Direct vs Regular comparison** — side-by-side TER drag on long-term compounding
- [ ] **Docker Compose setup** — single command to run the full stack in a container

---

## What I learned

The hardest part was not the metrics math — it was making the data pipeline reliable. AMFI's feed has inconsistent spacing. mfapi.in occasionally returns empty arrays for valid scheme codes. NSE's TRI API has different field names depending on the index. AUM data uses AMC names that don't match scheme names in the NAV feed, so `normaliseName()` in `scripts/` was a real project of its own — string normalisation, false-positive checks, match-rate audits across 9,100 entries.

The second hardest thing was the boot sequence. Getting from "server starts" to "data is ready to serve" in a way that's fast on warm restarts, correct on cold starts, and doesn't leave users confused about what's happening required building the three-tier cache system and the polling progress bar from scratch.

Everything else — the metrics, the API, the SPA routing — was straightforward once the data was clean.

---

<div align="center">

**Made by [Harleen Khanuja](https://www.linkedin.com/in/harleenkhanuja/)**

*Not financial advice. Always consult a SEBI-registered advisor before investing.*

</div>
