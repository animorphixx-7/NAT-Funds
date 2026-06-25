# 📊 Financial Intelligence Platform
### Mutual Fund Analytics & Performance Evaluation System

> A production-grade Node.js analytics engine that aggregates live data from AMFI, NSE/BSE, and RBI to compute institutional-quality risk and performance metrics across India's 9,100+ mutual fund schemes — with zero paid APIs and a zero-build-step frontend.

---

![Node.js](https://img.shields.io/badge/Node.js-20.x-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.x-000000?style=for-the-badge&logo=express&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES2022-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Chart.js](https://img.shields.io/badge/Chart.js-4.x-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
![Version](https://img.shields.io/badge/Version-2.0.0-brightgreen?style=for-the-badge)

---

## 📌 Project Overview

The **Financial Intelligence Platform** is a full-stack mutual fund analytics system designed to democratize institutional-grade financial analysis for retail investors and researchers in India. Most retail tools either display a star rating or hide advanced metrics behind expensive subscriptions. This platform changes that.

It integrates **six live data sources**, computes **12+ quantitative financial metrics** per fund, and delivers everything through a paginated REST API backed by a **three-tier caching system** — resulting in fast cold boots and near-instant warm restarts.

This project was developed as part of an internship engagement focused on data engineering, financial analytics, and full-stack platform architecture.

---

## 🎯 Project Objectives

1. Build an end-to-end data pipeline ingesting live AMFI, NSE/BSE, and RBI feeds
2. Implement institutional-grade financial metric computation (CAGR, Sharpe, Sortino, Beta, Alpha, Drawdown, Capture Ratios)
3. Design a three-tier caching architecture for high-throughput, low-latency performance
4. Deliver a responsive single-page frontend with fund search, filtering, comparison, and charting
5. Expose a clean REST API enabling programmatic access to all computed analytics
6. Provide admin endpoints for manual data refresh triggers
7. Establish a smoke-test suite covering all critical modules

---

## ✨ Features

### 📡 Data Ingestion
- Live NAV feed for **9,100+ AMFI schemes** (boot + on-demand refresh)
- Full per-scheme NAV history via `mfapi.in` (24-hour disk cache per scheme)
- AMFI Fund Performance API for AUM, benchmark map, SEBI Riskometer, and Information Ratios
- AMFI TER XLSX for Total Expense Ratio per scheme
- NSE/BSE Indices API for benchmark TRI history (Beta, Alpha, Capture Ratios)
- CCIL India (RBI 91-day T-bill) for risk-free rate computation

### 📐 Metric Computation Engine
| Category | Metrics |
|---|---|
| **Returns** | CAGR 1Y / 3Y / 5Y / Since Inception · Rolling Returns (1Y daily-step, 3Y monthly-step with percentile distribution) |
| **Risk** | Standard Deviation · Max Drawdown (with recovery date) · SEBI Riskometer |
| **Risk-Adjusted** | Sharpe Ratio · Sortino Ratio · Calmar Ratio |
| **Benchmark-Relative** | Beta · Jensen's Alpha · Information Ratio · R² · Upside/Downside Capture |
| **Peer Scoring** | Consistency Score 0–10 (percentile-normalised within sub-category) |
| **AMFI-Published** | Official IR (1Y / 3Y / 5Y / 10Y, Direct + Regular plans) |

### 🌐 REST API
- `GET /api/status` — Server readiness and loading progress
- `GET /api/categories` — Category/sub-category counts with optional filters
- `GET /api/funds` — Paginated fund list with filtering, sorting, and lazy metric evaluation
- `GET /api/fund/:schemeCode` — Single fund detail with live NAV sync and on-the-fly recalculation
- `GET /api/fund/:schemeCode/nav-history` — Sampled monthly NAV data for charting (1Y/3Y/5Y/max)
- `GET /api/compare` — Side-by-side comparison of 2–4 funds
- `GET /api/search` — Full-text search across scheme name and AMC

### 🔐 Security & Operations
- Helmet.js security headers (X-Content-Type-Options, X-Frame-Options, HSTS)
- Rate limiting: 200 requests/min/IP
- Optional admin authentication via `X-Admin-Secret` header
- Admin endpoints for manual TER, AUM, and TRI sync triggers
- Structured logging via Pino
- Daily cron jobs (09:00, 09:30, 09:31 IST) for automated data refresh

### 🖥️ Frontend
- Zero-build-step SPA (vanilla HTML + ES modules, no bundler needed)
- Fund exploration with multi-dimension filtering (type, sub-category, plan type, market cap)
- Fund detail page with interactive Chart.js NAV charts
- Side-by-side fund comparison view (2–4 funds)
- Responsive design with Tailwind CSS (CDN)
- Material Symbols iconography

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Runtime** | Node.js 20.x | Server-side JavaScript engine |
| **Framework** | Express 4.x | HTTP server and routing |
| **Security** | Helmet 8.x | HTTP security headers |
| **Rate Limiting** | express-rate-limit 8.x | API abuse prevention |
| **Logging** | Pino 10.x + pino-pretty | Structured JSON logging |
| **Scheduling** | node-cron 4.x | Automated daily data sync |
| **HTTP Client** | Axios 1.x | External API calls |
| **Excel Parsing** | ExcelJS 4.x + xlsx 0.18.x | AMFI TER XLSX ingestion |
| **Environment** | dotenv 17.x | Configuration management |
| **Frontend** | Vanilla JS (ES Modules) | Zero-build SPA |
| **Charts** | Chart.js 4.x (CDN) | NAV history and performance charts |
| **Styling** | Tailwind CSS (CDN) | Responsive utility-first styling |
| **Icons** | Material Symbols (CDN) | Icon system |
| **Testing** | Node.js built-in test runner | Smoke test suite |

---

## 📁 Folder Structure

```
financial-intelligence-platform/
│
├── server.js                    # Application entry point
├── package.json                 # Dependencies and scripts
├── nodemon.json                 # Dev server configuration
├── .gitignore
│
├── boot/                        # Boot sequence and data helpers
│   ├── startup.js               # Full boot orchestration (TER → AUM → TRI → NAV → metrics)
│   └── dataHelpers.js           # TER/AUM/IR/Riskometer enrichment helpers
│
├── routes/                      # Express route handlers (HTTP only, no business logic)
│   ├── api.js                   # All /api/* REST endpoints
│   ├── admin.js                 # Protected /admin/* sync endpoints
│   └── ter.js                   # TER-specific routes
│
├── services/                    # Core business logic layer
│   ├── metricsCalculator.js     # 1,200-line pure function metrics engine
│   ├── fundPerformanceService.js# AMFI Fund Performance API integration
│   ├── triService.js            # NSE/BSE TRI benchmark data integration
│   ├── terService.js            # AMFI TER XLSX ingestion
│   ├── dataFetcher.js           # mfapi.in NAV history fetching + caching
│   ├── amfiParser.js            # AMFI NAV text feed parser
│   └── riskFreeRate.js          # RBI 91-day T-bill rate fetching
│
├── shared/                      # Shared application state and utilities
│   ├── appState.js              # Single source of truth for in-memory state
│   └── logger.js                # Pino logger configuration
│
├── public/                      # Frontend SPA (served statically)
│   ├── index.html               # Shell HTML + CDN imports
│   ├── styles.css               # Custom CSS
│   └── js/
│       ├── api.js               # Frontend API client
│       ├── charts.js            # Chart.js wrappers
│       ├── compare.js           # Compare page logic
│       ├── constants.js         # Shared constants
│       ├── formatters.js        # Number/date formatters
│       ├── init.js              # App initialisation
│       ├── router.js            # Client-side routing
│       ├── search.js            # Search UI logic
│       ├── state.js             # Frontend state management
│       ├── ui.js                # UI utility functions
│       └── views/
│           ├── home.js          # Home / fund explorer view
│           ├── explore.js       # Explore / filter view
│           ├── fund-detail.js   # Single fund detail view
│           └── compare.js       # Fund comparison view
│
├── data/                        # Persisted JSON data (auto-generated)
│   ├── aum-data.json            # Daily AUM (Crores) per fund
│   ├── benchmark-data.json      # Fund → benchmark name mapping
│   ├── ir-data.json             # AMFI Information Ratios
│   ├── riskometer-data.json     # SEBI Riskometer labels
│   ├── ter-data.json            # Total Expense Ratios
│   └── tri-data.json            # Benchmark TRI history (~14MB)
│
├── data-sources/                # Raw Excel source files
│   └── aum_data/                # AMFI AUM xlsx files (historical)
│
├── sql/                         # Database schema and analytical queries
│   ├── schema.sql               # Full DDL — all CREATE TABLE statements
│   └── queries.sql              # Analytical SQL queries
│
├── docs/                        # Project documentation
│   ├── Project_Report.md        # Full 15–20 page internship report
│   ├── Architecture.md          # System architecture with Mermaid diagrams
│   ├── API_Documentation.md     # Complete REST API reference
│   └── Database_Design.md       # Schema, ER diagram, design decisions
│
├── scripts/                     # Utility / normalisation scripts
│   ├── audit_aum_norm.js        # AUM data normalisation audit
│   └── test_normalise.js        # Normalisation test helpers
│
├── dev-tools/                   # Internal development and debugging utilities
│   └── ...                      # (not part of production runtime)
│
└── tests/                       # Automated test suite
    └── smoke.test.js            # Node.js built-in smoke tests (9 tests, ~0.5s)
```

---

## ⚡ At a Glance

| Metric | Value |
|---|---|
| AMFI schemes tracked | **9,100+** |
| Quantitative metrics per fund | **12+** |
| Live data sources integrated | **6** |
| Caching tiers | **3** (persistent JSON · per-scheme disk · processed snapshot) |
| Funds with full NAV history + all metrics | **~3,000** |
| API rate limit | **200 req/min/IP** |
| Smoke test suite runtime | **~0.5s** (9 tests, zero network calls) |
| Build steps required | **0** |

---

## 🚀 Installation Guide

### Prerequisites
- **Node.js** v18 or higher (v20 LTS recommended)
- **npm** v9 or higher
- Internet access (live data from AMFI, mfapi.in, NSE/BSE, RBI)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/<your-username>/financial-intelligence-platform.git
cd financial-intelligence-platform
```

### Step 2 — Install Dependencies

```bash
npm install
```

### Step 3 — Configure Environment (Optional)

```bash
cp .env.example .env
```

Edit `.env` as needed:

```env
PORT=3001                   # Server port (default: 3001)
ADMIN_SECRET=your_secret    # Admin endpoint protection (leave blank to disable)
CORS_ORIGIN=                # CORS origin (leave blank — same-origin by default)
LOG_LEVEL=info              # Pino log level: trace | debug | info | warn | error
```

### Step 4 — Start the Server

```bash
# Production
npm start

# Development (auto-restart with nodemon)
npm run dev
```

### Step 5 — Access the Application

Open your browser at: **http://localhost:3001**

The server will display a boot sequence log showing data loading progress. On first boot, it fetches fresh data from all six sources (takes 30–60 seconds). Subsequent starts use the three-tier cache and are near-instant.

### Step 6 — Run Tests

```bash
npm test
```

Expected output: 9 passing smoke tests in ~0.5s, zero network calls.

---

## 📸 Screenshots

> _Screenshots to be added after UI deployment. The following views are available in the application:_

| View | Description |
|---|---|
| **Home / Explorer** | Paginated fund list with multi-dimension filters (type, sub-category, plan, market cap), sortable columns, and search |
| **Fund Detail** | Full metrics card, interactive NAV chart (1Y/3Y/5Y/Max), rolling return distribution, SEBI Riskometer badge |
| **Fund Comparison** | Side-by-side comparison of 2–4 funds across all computed metrics with cross-category warnings |
| **Search** | Real-time full-text search across 9,100+ scheme names and AMC names |

---

## 🔌 API Overview

Base URL: `http://localhost:3001/api`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/status` | Server readiness and boot progress |
| `GET` | `/categories` | Category/sub-category fund counts |
| `GET` | `/funds` | Paginated fund list (filters: type, subCategory, planType, optionType, marketCap, search, sortBy, order, page, limit) |
| `GET` | `/fund/:schemeCode` | Single fund with live NAV sync and on-the-fly metrics |
| `GET` | `/fund/:schemeCode/nav-history` | Monthly NAV series for charting (period: 1y/3y/5y/max) |
| `GET` | `/compare?codes=A,B,C` | Side-by-side comparison of 2–4 funds |
| `GET` | `/search?q=query` | Full-text search (min 2 chars, top 20 results) |

**Admin Endpoints** (require `X-Admin-Secret` header if `ADMIN_SECRET` is set):

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/admin/sync-ter` | Trigger full TER sync from AMFI |
| `GET` | `/admin/sync-aum` | Trigger AUM + Riskometer refresh |
| `GET` | `/admin/sync-tri` | Refresh all benchmark TRI indices |

> Full API documentation with request/response schemas is in [`docs/API_Documentation.md`](docs/API_Documentation.md).

---

## 📊 Data Sources

| Source | Data Provided | Refresh Cadence |
|---|---|---|
| `amfiindia.com` NAV feed | Live NAV for ~9,100 schemes | Boot + on-demand |
| `mfapi.in` | Full per-scheme NAV history | Per-scheme disk cache (24h TTL) |
| AMFI Fund Performance API | AUM, benchmark map, AMFI IR, SEBI Riskometer | Daily cron — 09:30 IST |
| AMFI TER XLSX | Total Expense Ratio per scheme | Daily cron — 09:00 IST |
| NSE/BSE Indices API | Benchmark TRI history | Daily cron — 09:31 IST |
| CCIL India (RBI T-bill) | 91-day risk-free rate | 24h disk cache |

---

## 🔭 Future Scope

| Feature | Description |
|---|---|
| **Portfolio Simulation** | Allow users to build hypothetical portfolios and track blended performance |
| **SIP Returns Calculator** | Compute actual SIP returns with dividend reinvestment using historical NAV |
| **Goal-Based Investing** | Map funds to financial goals (retirement, education, home) with required SIP computation |
| **PostgreSQL Migration** | Migrate JSON flat-file storage to a relational database for query flexibility |
| **PDF Report Export** | Generate downloadable PDF fact sheets per fund using Puppeteer |
| **Alerts Engine** | NAV breach / drawdown threshold push notifications |
| **Mobile App** | React Native frontend consuming the existing REST API |
| **ML Risk Scoring** | Machine learning model for forward-looking fund risk classification |
| **Direct Plan Advisor** | Personalised direct vs. regular plan cost comparison with long-term impact analysis |

---

## 👤 Author

**Kaushik Jadhav**
Data Analyst Intern

📧 (kaushikjadhav77@gmail.com)
🔗 [LinkedIn](https://www.linkedin.com/in/kaushik-jadhav-7a2a9b286)
🐙 [GitHub](https://github.com/animorphixx-7)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgements

- [AMFI India](https://www.amfiindia.com/) for open NAV data feeds and Fund Performance API
- [mfapi.in](https://www.mfapi.in/) for historical NAV history API
- [NSE India](https://www.nseindia.com/) and [BSE India](https://www.bseindia.com/) for benchmark TRI data
- [RBI / CCIL India](https://www.ccilindia.com/) for 91-day T-bill risk-free rate

---

> *"The goal of financial analytics is not to predict the future — it is to understand the present well enough to make better decisions."*

