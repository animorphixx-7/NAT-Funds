# NAT Funds 📊

> A full-stack mutual fund analytics platform I built because I got tired of jumping between Tickertape, Value Research, and Groww just to compare a few funds properly.

NAT Funds pulls live data from AMFI, NSE/BSE, and RBI, computes 12+ institutional-grade risk/return metrics from scratch, and serves everything through a clean REST API and a no-build-step SPA frontend. ~9,100 schemes. Zero paywalls.

---

## Why I built this

Retail investors in India have surprisingly bad tooling. Most platforms either hide the good metrics behind a subscription or just show you the AMFI-published numbers without any context. I wanted to understand how Sharpe, Sortino, Beta, and Jensen's Alpha were actually calculated — not just displayed — so I built the whole thing myself, from parsing the AMFI NAV feed to writing a two-pass O(n) max drawdown scanner.

The metrics in `services/metricsCalculator.js` (~1,200 lines, pure functions) are the real core of this project. Everything else is plumbing.

---

## What it does

**Browse** ~9,100 AMFI schemes across categories, filter by plan/option type, sort by any metric, paginate.

**Compare** up to 4 funds side-by-side with a full breakdown across returns, risk, and benchmark-relative metrics.

**Deep-dive** into any fund: rolling return distributions (1Y daily-step, 3Y monthly-step), drawdown timeline, upside/downside capture ratios, consistency score.

**Search** live across scheme names and AMCs.

---

## Metrics computed

| Category | What's in there |
|---|---|
| Returns | CAGR 1Y / 3Y / 5Y / since inception, rolling return distributions |
| Risk | Annualised Std Dev, Max Drawdown (peak → trough → recovery), SEBI Riskometer |
| Risk-adjusted | Sharpe, Sortino, Calmar |
| Benchmark-relative | Beta, Jensen's Alpha, Information Ratio, R², Upside/Downside Capture |
| Fund metadata | AUM (₹ Cr), TER, Benchmark name, Plan type |
| Peer scoring | Consistency Score (0–10, percentile-ranked within sub-category) |

A few things worth noting on methodology:
- **Sharpe/Sortino** use the live RBI 91-day T-bill YTM as the risk-free rate (fetched and cached daily), not a hardcoded 6%.
- **Beta and Alpha** use real benchmark TRI history pulled from NSE/BSE — not approximate index price data.
- **IDCW plans**: rolling returns reflect NAV movement only, matching Tickertape/INDmoney methodology (dividends paid out are not added back).
- **Calmar** returns `null` if 3Y CAGR is unavailable or max drawdown is zero — no fake numbers.

---

## Stack

**Backend**: Node.js + Express  
**Frontend**: Vanilla JS (ES modules, no bundler, no framework)  
**Data**: AMFI NAV feed · mfapi.in · NSE/BSE Indices API · RBI CCIL · AMFI TER XLSX  
**Caching**: Three-tier disk cache (persistent JSON stores + per-scheme NAV cache + computed fund list)  
**Logging**: Pino structured logger  
**Security**: Helmet, express-rate-limit (200 req/min/IP)

---

## Architecture at a glance

```
server.js                      ← Express setup, security headers, route mounting, boot trigger
│
├── shared/
│   ├── appState.js            ← Single source of truth for all in-memory state
│   └── logger.js              ← Pino logger (level from LOG_LEVEL env)
│
├── boot/
│   ├── startup.js             ← Full boot sequence: parse → fetch → compute
│   └── dataHelpers.js         ← Per-fund enrichment (TER, AUM, AMFI IR, Benchmark, Riskometer)
│
├── routes/
│   ├── api.js                 ← /api/* REST endpoints
│   ├── admin.js               ← /admin/* sync endpoints (auth-protected)
│   └── ter.js                 ← /ter/:schemeCode lookup
│
├── services/
│   ├── amfiParser.js          ← Parses live AMFI NAV text feed; selects top funds
│   ├── dataFetcher.js         ← NAV history from mfapi.in + disk cache
│   ├── metricsCalculator.js   ← All metric math (~1,200 lines, pure functions)
│   ├── fundPerformanceService.js ← AUM, Benchmark, AMFI IR, SEBI Riskometer
│   ├── terService.js          ← TER from AMFI XLSX (ExcelJS)
│   ├── triService.js          ← Benchmark TRI from NSE/BSE
│   └── riskFreeRate.js        ← 91-day T-bill from RBI (24h cache)
│
├── data/                      ← Persistent JSON stores (committed)
├── cache/                     ← Gitignored; auto-populated at runtime
├── public/                    ← SPA (vanilla HTML + JS modules, no build step)
│   └── js/
│       ├── views/             ← home, explore, fund-detail, compare
│       └── ...                ← state, api wrapper, formatters, router, charts
├── tests/
│   └── smoke.test.js          ← 9 smoke tests, ~0.5s, no network
└── dev-tools/                 ← 19 diagnostic scripts, never imported by server
```

---

## Caching strategy

Cold boot on a fresh clone hits the network. After that, three cache tiers kick in:

**`data/*.json`** — Persistent stores for TER, AUM, TRI, benchmarks, AMFI IR, riskometers. Committed to the repo. Refreshed by cron jobs and admin endpoints. Benchmark data has a 90-day TTL (SEBI mandates semi-permanent benchmark assignments), everything else is daily.

**`cache/nav_<schemeCode>.json`** — Per-scheme NAV history from mfapi.in. 24-hour TTL. Git-ignored. Auto-populates on first fetch, fully rebuilt within 24h.

**`cache/processed_funds.json`** — The full parsed+computed fund list with all metrics applied. 24-hour TTL. On a warm restart (< 24h), the server loads this directly and skips the entire parse → fetch → compute cycle. This is what makes restarts fast in production.

**`cache/ter-parsed.json`** — Pre-parsed TER data. Avoids re-downloading the AMFI TER XLSX on every restart.

---

## Getting started

### Prerequisites
- Node.js 18+
- Internet access on first boot (fetches live AMFI data)

### Install & run

```bash
git clone https://github.com/harleenkhanuja/NAT-Funds.git
cd NAT-Funds
npm install
npm start        # production
npm run dev      # development with nodemon
npm test         # 9 smoke tests, ~0.5s, no network required
```

On first boot the server:
1. Loads TER from `data/ter-data.json` (or syncs from AMFI if missing)
2. Loads AUM, benchmarks, AMFI IR, riskometers from `data/` (or syncs if stale)
3. Fetches the RBI risk-free rate
4. Checks for a fresh `cache/processed_funds.json` — if < 24h old, loads it and skips 5–7
5. Parses the live AMFI NAV feed (~9,100 schemes)
6. Batch-fetches NAV history for top ~3,000 schemes (10 concurrent, disk-cached)
7. Computes all metrics in memory, saves `processed_funds.json`
8. Sets `dataReady = true` — starts serving the SPA

The frontend shows a live progress bar (polling `/api/status`) during steps 5–7 so you're not staring at a blank screen.

### Environment variables

```env
PORT=3001              # HTTP port (default: 3001)
ADMIN_SECRET=          # If set, /admin/* requires X-Admin-Secret header
CORS_ORIGIN=           # Leave empty — frontend is served by this same server
LOG_LEVEL=info         # Pino levels: trace | debug | info | warn | error | fatal
```

---

## API reference

### Public endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/status` | Server readiness + boot progress |
| `GET` | `/api/categories` | Category/sub-category counts; `?planType=` + `?optionType=` |
| `GET` | `/api/funds` | Paginated fund list with filtering + sorting |
| `GET` | `/api/fund/:schemeCode` | Single fund detail with live NAV sync + on-the-fly metric recalc |
| `GET` | `/api/fund/:schemeCode/nav-history` | Sampled monthly NAV for charting; `?period=1y|3y|5y|max` |
| `GET` | `/api/compare?codes=...` | Side-by-side comparison for 2–4 scheme codes |
| `GET` | `/api/search?q=...` | Full-text search across scheme name + AMC (top 20) |
| `GET` | `/ter/:schemeCode` | TER lookup for a single scheme |

Query params for `/api/funds`:

| Param | Example | Notes |
|---|---|---|
| `type` | `Equity` | Filter by fund type |
| `subCategory` | `Large Cap` | Comma-separated for multiple |
| `planType` | `Direct` | `Direct` or `Regular` |
| `optionType` | `Growth` | `Growth` or `IDCW` |
| `search` | `HDFC` | Full-text across name + AMC |
| `sortBy` | `cagr3y` | Supports nested objects |
| `order` | `desc` | `asc` or `desc` |
| `page` | `1` | 1-indexed |
| `limit` | `20` | Max 100 |

### Admin endpoints (require `X-Admin-Secret` header if env var is set)

```
GET /admin/sync-ter   → Re-download AMFI TER XLSX; rebuild ter-data.json
GET /admin/sync-aum   → Refresh AUM + AMFI IR + Riskometer from AMFI Fund Performance API
GET /admin/sync-tri   → Refresh all benchmark TRI histories from NSE/BSE
```

Cron schedule: TER at 09:00 IST, AUM at 09:30 IST, TRI at 09:31 IST.

---

## Data sources

| Source | What I pull | Refresh cadence |
|---|---|---|
| `amfiindia.com` NAV text feed | Live NAV for ~9,100 schemes | On demand / boot |
| mfapi.in | Per-scheme historical NAV (full history) | 24h disk cache per scheme |
| AMFI Fund Performance API | AUM, fund→benchmark mapping, AMFI IR (1Y/3Y/5Y/10Y), SEBI Riskometer | Daily at 09:30 IST |
| AMFI TER XLSX | Total Expense Ratio per scheme | Daily at 09:00 IST |
| NSE / BSE Indices API | Benchmark TRI history | Daily at 09:31 IST |
| CCIL India (RBI) | 91-day T-bill YTM (risk-free rate) | 24h cache, daily at boot |

---

## Dev tools & scripts

`dev-tools/` has 19 standalone diagnostic scripts — safe to run at any time, never imported by the server.

```bash
node dev-tools/test-api.js          # Basic API smoke test
node dev-tools/test-metrics.js      # Metrics calculator unit checks
node dev-tools/diagnose.js          # Data-coverage diagnostic report
node dev-tools/test-benchmark.js    # TRI benchmark lookup check
```

`scripts/` has data-quality audit tools for AUM normalisation:

```bash
node scripts/audit_aum_norm.js      # AUM match-rate audit across all 9,100+ funds
node scripts/test_normalise.js      # Unit tests for normaliseName() + false-positive check
```

`dev-tools/app_original.js` is the original monolithic frontend (~88 KB) I started with. It's kept for reference — the live app loads exclusively from `public/js/` modules.

---

## A few things I learned building this

- Parsing the AMFI NAV text feed is deceptively annoying. The format uses `|`-delimited lines with category headers interspersed — you have to track state as you scan through it.
- Getting Beta and Alpha right required pulling *Total Return Index* data from NSE/BSE, not just closing prices. Regular index prices don't account for dividends, so the benchmark return is understated.
- The three-tier cache design was something I figured out after realising a cold boot was taking 3–4 minutes on a cheap VPS. Warm restarts now take under 5 seconds.
- The Calmar Ratio edge cases were subtle — you can get a 0% max drawdown on some very short-history funds, which would cause a divide-by-zero. Returning `null` explicitly felt cleaner than returning `Infinity`.

---

## Notes

- No build step — the frontend is plain HTML + vanilla JS ES modules loaded in strict dependency order in `index.html`. No webpack, no Vite, no React.
- All financial calculations are pure functions with no side effects. `metricsCalculator.js` can be imported and tested in isolation.
- Rate limiting is set to 200 req/min/IP at the Express layer.
- The `cache/` directory is git-ignored and auto-populates on first boot.
- `data/*.json` files are committed so the app works immediately on clone without waiting for a full re-sync.

---

## Tech used

`Node.js` · `Express` · `Axios` · `ExcelJS` · `node-cron` · `Helmet` · `Pino` · `Chart.js` (frontend) · `Vanilla JS ES Modules`

---

*Built by Harleen Khanuja*
