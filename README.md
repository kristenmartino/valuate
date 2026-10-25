# Valuate — SEC filing valuation workspace

Valuate is a portfolio prototype for reviewing extracted financials and exploring valuations under selected assumptions. Implementation was AI-assisted; the [case study](https://kristenmartino.ai/work/valuate) describes the product framing, financial requirements, validation work, and limits.

| Resource | Link |
|---|---|
| Live app | [valuate.kristenmartino.ai](https://valuate.kristenmartino.ai) |
| Case study | [Valuate](https://kristenmartino.ai/work/valuate) |
| Frontend | [valuate-web](https://github.com/kristenmartino/valuate-web) — Next.js on Vercel |
| Backend | [valuate-api](https://github.com/kristenmartino/valuate-api) — FastAPI and LangGraph on Railway |

## Extraction and review

Choose one of 18 sample tickers or enter a US ticker with a SEC 10-K. The app uses a saved company record when available. Uncached companies require filing retrieval and extraction, which can fail when required values remain missing. Saved records do not establish that the filing is current.

SEC XBRL company facts provide the first extraction path. Claude fills gaps from filing text; supported calculations supply some remaining fields. Line items identify SEC XBRL, AI extraction, calculation, or user correction as their source. Expand source details for an XBRL concept, AI-provided excerpt, or calculation explanation where available. XBRL tags do not apply to every source, and excerpts require manual checking.

Automated checks flag extraction scores below 0.80 and balance-sheet discrepancies above 0.5%. Scores are heuristics, not calibrated probabilities of correctness. No flags does not mean a person verified the figures. Reviewers can replace flagged values, choose a specific line for a balance-sheet reconciliation, and optionally add an excerpt. Checks run again after saving. Corrections retain the current value and source label, without reviewer identity or chronological edit history.

Balance-sheet reconciliation keeps parent shareholders' equity and noncontrolling interests separate, adds them when required, and avoids double-counting equity already tagged as consolidated. Prologis's missing component can be backfilled from the same annual filing while preserving its existing amounts, correction notes, and valuation. Unknown equity coverage or unavailable source facts still require review.

CRM's statement mapping uses noncurrent debt separately from current maturities, total cash-flow D&A rather than a fixed-asset component, and an explicit combined payables/accruals label. A targeted filing-backed repair updates older machine-extracted records while preserving user-corrected values and supporting notes. These corrections affect valuation inputs; they do not certify every extracted field.

CRM's revenue breakdown uses five tagged subscription/support offerings plus professional services and other. The categories reconcile to total revenue without adding the subscription subtotal again. The panel calls these revenue categories, rather than separate operating segments, and the consolidated valuation is unchanged.

## Valuation models

| Category | Simplified model |
|---|---|
| Standard | Five-year FCFF DCF with terminal growth |
| Bank | Gordon dividend discount |
| Insurer | Justified price-to-book |
| REIT | Gordon growth using approximate FFO: net income plus D&A |
| Energy | Fixed ten-year FCFF projection without terminal value |

Berkshire (`BRK.B` / `BRK-B`, also Class A) uses the simplified justified-P/B formula with consolidated GAAP earnings and Berkshire shareholders' equity, not a sum-of-the-parts valuation. The annual filing supplies class-specific average equivalent shares and dimensioned premiums/cash; the workspace discloses that share basis. Investment gains can distort ROE, and average shares differ from year-end shares.

Statement labels distinguish source-specific investment and policy-liability components from totals. Berkshire cash retains Treasury bills classified as cash equivalents and excludes separately reported restricted cash and short-term investments in Treasury bills.

Energy growth changes projected revenue. The model does not estimate company-specific reserve life, production volumes, or commodity prices. The REIT calculations are simplified estimates, not company-reported FFO/AFFO. Unclassified companies fall back to standard DCF, which may not fit their business.

Assumption controls include the starting values even when they fall outside the usual slider ranges, as TXN's 22.58% capex/revenue assumption does. **Restore starting assumptions** returns all inputs to the full-precision values loaded when the workspace opened and recalculates the scenario. It changes scenario inputs without replacing saved financial figures.

Monte Carlo output is a distribution under sampled assumptions, not a calibrated stock-price forecast. The chart defaults to values between the 10th and 90th percentiles, discloses the counts outside that window, and offers the full range. Summary statistics use all completed simulations. Sensitivity grids and DCF-implied multiples appear for standard and energy models. Bank, insurer, and REIT comps tables show market context without a model-implied row.

## Evidence and storage limits

The [recorded extraction baseline](https://github.com/kristenmartino/valuate-api#extraction-eval-baseline), last refreshed May 30, 2026, reports 39/39 XBRL fields and 3/4 Claude fields within ±0.5% of pinned reference values. The combined 42/43 result spans 11 filers. The AI sample is small; this historical baseline does not establish general extraction accuracy. Deterministic tests cover specific valuation, correction, and chart-data behaviors, rather than every filer or model assumption.

Company records use Postgres when `DATABASE_URL` is configured; otherwise they use a process-local cache that is lost on restart. The frontend proxy keeps its configured backend credential server-side. Filing freshness, broader evaluation, and correction history need further work before a production research workflow.
