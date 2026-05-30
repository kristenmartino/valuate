# Valuate — Live AI-augmented SEC 10-K valuation workspace

Valuate is a live, deployed AI-augmented valuation workspace that ingests SEC 10-K filings, extracts financials with XBRL plus LLM fallback, surfaces line-item provenance, and runs industry-aware valuation models with Monte Carlo, sensitivity analysis, and trading comps.

### **▶ [Live app — valuate.kristenmartino.ai](https://valuate.kristenmartino.ai)**  ·  [Case study](https://kristenmartino.ai/work/valuate)

| | |
|---|---|
| **Live app** | https://valuate.kristenmartino.ai |
| **Case study** | https://kristenmartino.ai/work/valuate |
| **Frontend** | [kristenmartino/valuate-web](https://github.com/kristenmartino/valuate-web) — Next.js 16, deployed on Vercel |
| **Backend** | [kristenmartino/valuate-api](https://github.com/kristenmartino/valuate-api) — FastAPI + LangGraph, deployed on Railway |
| **Recent hardening** | [valuate-api#6](https://github.com/kristenmartino/valuate-api/pull/6) (industry-aware overrides + FCFF valuation tests) · [valuate-web#1](https://github.com/kristenmartino/valuate-web/pull/1) (valuation request-ordering guard) |

## What it does

Pick one of 18 curated tickers (or search any SEC filer), and Valuate:

1. **Ingests** the company's latest 10-K from SEC EDGAR.
2. **Extracts** the financial line items two ways — structured XBRL company-facts first (Track A), then a Claude fallback for anything XBRL left blank (Track B) — and shows the **source, confidence, and XBRL tag for every number**.
3. Lets a reviewer **correct any flagged value** (human-in-the-loop override).
4. Runs an **industry-aware valuation** — standard FCFF DCF, bank dividend-discount, insurer justified price-to-book, REIT FFO-multiple, or energy reserve-life — with a **10,000-iteration Monte Carlo**, a **sensitivity grid**, and **trading comps** including a DCF-implied-multiples cross-check.

## Why it's interesting

- **Cell-level provenance.** Every extracted figure carries where it came from and how confident the system is — the extraction is auditable, not a black box.
- **Five industry-specific valuation models**, each implementing the textbook closed form, dispatched automatically by SIC code.
- **Valuation math validated** through live-site QA, deterministic unit tests, and adversarial code review — the standard FCFF model is hand-checked against live outputs (FCFF buildup, terminal value, EV→equity bridge, per-share fair value), and the industry models are tested against closed-form formulas.
- **Production engineering** — same-origin proxy (the auth token never reaches the browser), structured request logging with correlation IDs, Sentry, a `/version` endpoint exposing commit SHA + prompt hash, and Postgres-backed persistence.

## Scope & honesty

Valuate is a portfolio-grade product prototype, not production investment software. Extraction is validated on a sample of filers, not exhaustively across all SEC registrants; the highest-confidence experience is the curated universe and the five explicitly modeled industries. Companies outside those industries fall back to the standard FCFF model, which may not be appropriate for them. Nothing here is investment advice.

## Architecture at a glance

```
Browser ─▶ valuate-web  (Next.js 16 · Vercel)
              │  same-origin /api/* rewrite — auth token injected server-side
              ▼
           valuate-api  (FastAPI + LangGraph · Railway)
              │  ingest ─▶ XBRL (Track A) ─▶ Claude (Track B) ─▶ validate
              ▼
           SEC EDGAR   +   Anthropic Claude
```

Each repo's README is the working reference for its code; the [case study](https://kristenmartino.ai/work/valuate) has the full design narrative.
