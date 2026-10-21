# Securities Analysis

A personal, web-based securities research platform for analysing listed companies, estimating intrinsic value, ranking potentially undervalued stocks, and researching options strategies.

The aim is to make investment research **repeatable, explainable, and data-driven**—not to provide automated trading signals or execute trades.

> **Project status:** Planning / initial setup. This repository does **not** yet contain a runnable application. Features and commands described as *planned* are not implemented.

## Product vision

Securities Analysis will let an authenticated user:

- **Find securities** by ticker or company name and inspect delayed/end-of-day prices and company fundamentals.
- **Estimate intrinsic value** over a user-defined investment horizon, with configurable expected annual return, growth assumptions, and margin of safety.
- **Understand each valuation** through transparent inputs, calculation steps, data timestamps, and reproducible saved results.
- **Screen and rank stocks** by estimated undervaluation against a consistent set of model assumptions.
- **Save favourites and watchlists** for further research.
- **Research covered calls** and eventually other options strategies, using appropriately timestamped options-chain data (post-MVP).

The first release focuses on equities and fundamental valuations. Real-time data, automated order execution, an LLM chatbot, and complex options strategies are not required for the MVP.

## Planned MVP

| Capability | MVP scope |
| --- | --- |
| Stock data | Batch-updated company information, prices, and fundamental metrics |
| Search | Find a stock by symbol or company name; inspect its details |
| Intrinsic valuation | Earnings-based model with user-defined timeframe and required annual return |
| Assumptions | Adjustable growth, valuation multiple, discount rate, and margin of safety |
| Results | Estimated fair value, potential upside/downside, purchase threshold, and explanation |
| Screener | Rank a supported stock universe under a chosen, consistent assumptions preset |
| Watchlists | Personal favourites and saved stocks |
| Accounts | Private login; no public registration |
| Data history | Source, effective financial dates, and inputs saved alongside valuation results |

**Next phases:** free-cash-flow DCF and sensitivity analysis; covered-call research; additional options strategies; portfolio insights; optional LLM-assisted exploration.

## Technology decisions

| Layer | Planned technology | Why |
| --- | --- | --- |
| Backend | Django (Python) | Mature framework with authentication, ORM, migrations, and templating |
| Frontend | Django templates + HTMX where useful | Server-rendered experience without a separate frontend application |
| Database | PostgreSQL | Relational storage for securities, time-series snapshots, valuation runs, and watchlists |
| Local development | Docker Compose | Reproducible application and database environment |
| Initial production hosting | Single VPS with Docker Compose | Cost-controlled deployment and straightforward operations |
| Web server | Gunicorn behind a TLS-terminating reverse proxy (Caddy proposed) | HTTPS, application serving, and proxying |
| Data processing | Django management commands + scheduled jobs | Simple batch ingestion/screening without a distributed task queue |
| Financial-data access | Provider adapter(s), to be evaluated | Avoid tying valuation logic to one upstream API or scraping method |

React, Redis/Celery, Kubernetes, and microservices are **not** initial requirements. They can be introduced if real product needs justify their complexity.

See [architecture.md](architecture.md) for the deployment model, logical components, data conventions, security baseline, and decisions still open.

## Planned application flow

1. A scheduled job retrieves supported securities' prices and financial fundamentals from approved data sources.
2. Data is normalized and stored in PostgreSQL with original source and availability dates.
3. A signed-in user searches for a company, chooses a valuation model, and adjusts assumptions.
4. The valuation engine computes an explainable fair-value estimate and saves a reproducible snapshot.
5. The screener compares eligible companies using the **same model and assumptions** and ranks estimated upside/downside.
6. The user saves candidates to a watchlist for follow-up research.

Screening will use locally stored data rather than making a live third-party request for every company on every page load.

## Development and deployment

The project will support **local Docker Compose development** and **private hosting on a small VPS**. The first infrastructure issue is [#2 — Bootstrap dockerized Django and PostgreSQL](https://github.com/haberrj/securities-analysis/issues/2).

Once implemented, contributor documentation will cover:

- Prerequisites (Docker Engine and Docker Compose plugin).
- Example environment configuration with no committed credentials.
- Starting services, running migrations, creating an initial user, and running tests.
- Launching scheduled data updates and validating their status.
- Deploying to the VPS, obtaining HTTPS certificates, backing up the database, and restoring it.

**There are no working startup commands yet.** The README will be updated with verified commands when the infrastructure is implemented.

## Reliability and data principles

- **Data is not necessarily real-time.** Display quote dates, last-update times, financial periods, and provider information clearly.
- **Intrinsic value is an estimate, not a fact.** Surface assumptions, uncertainty, and sensitivity rather than presenting an unconditional buy/sell recommendation.
- **Safety margin is separate from fair value.** Display fair value and the discounted target purchase price separately.
- **Reproducibility matters.** Preserve model version, input data, assumptions, currency, and computation time for each saved valuation.
- **Use consistent units and dates.** Do not mix incompatible currencies, share bases, fiscal periods, or financial-data vintages.
- **Only ingest permitted data.** Data providers, scraping constraints, licences, and rate limits must be evaluated before implementation.
- **No trading permissions are required.** Future brokerage connections, if any, should begin as read-only.

## Roadmap / issues

Development is tracked in [GitHub Issues](https://github.com/haberrj/securities-analysis/issues).

1. **Foundation:** [architecture and MVP (#1)](https://github.com/haberrj/securities-analysis/issues/1), [Django/PostgreSQL Docker setup (#2)](https://github.com/haberrj/securities-analysis/issues/2), security and tests.
2. **Market data:** [schema (#3)](https://github.com/haberrj/securities-analysis/issues/3), [provider evaluation (#4)](https://github.com/haberrj/securities-analysis/issues/4), [ingestion (#5)](https://github.com/haberrj/securities-analysis/issues/5).
3. **Valuation and discovery:** [earnings-based valuation (#6)](https://github.com/haberrj/securities-analysis/issues/6), [saved runs (#8)](https://github.com/haberrj/securities-analysis/issues/8), [search (#9)](https://github.com/haberrj/securities-analysis/issues/9), [rankings (#10)](https://github.com/haberrj/securities-analysis/issues/10), [watchlists (#11)](https://github.com/haberrj/securities-analysis/issues/11), [dashboard (#12)](https://github.com/haberrj/securities-analysis/issues/12).
4. **Research expansion:** [DCF (#7)](https://github.com/haberrj/securities-analysis/issues/7), [options-data feasibility (#15)](https://github.com/haberrj/securities-analysis/issues/15), [covered-call research (#16)](https://github.com/haberrj/securities-analysis/issues/16).

## Security and scope

The intended production environment is an internet-accessible VPS, but the **application itself is private**: authenticated users only, with public registration disabled. HTTPS, secure cookies, login rate limiting, firewall rules, and off-server backups are part of the production baseline—not optional afterthoughts.

A small set of public endpoints (such as a health check, if needed) may be exposed deliberately; private financial pages and data must not be exposed anonymously.

## Disclaimer

This is a personal research and analytics tool. Financial information can be missing, incorrect, stale, or subject to corporate actions. All valuations and options calculations depend on assumptions and must be independently reviewed. The project does not provide investment advice or guarantee returns.
