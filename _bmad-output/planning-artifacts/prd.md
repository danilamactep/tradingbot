---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success, step-01b-continue, step-04-journeys]
classification:
  projectType: cli_tool
  domain: fintech
  complexity: high
  projectContext: greenfield
  complianceNote: personal single-user tool — standard fintech compliance (KYC/AML/PCI-DSS) does not apply
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-tradingbot-2026-03-07.md
workflowType: 'prd'
---

# Product Requirements Document - tradingbot

**Author:** Daniel
**Date:** 2026-03-11

---

## Executive Summary

tradingbot is a personal, offline Python CLI toolkit for a single developer-trader. It solves the accountability gap that causes systematic trading losses: without measuring the cost of emotional overrides, they cannot be reduced. The system recommends trades from a versioned strategy, records every human decision including deviations from recommendations, and surfaces the P&L delta between what Daniel did and what the system would have done. The nightly review is the core product — all other components exist to feed it or learn from it.

The system earns the right to execute autonomously only after measurable proof that it outperforms human decision-making over time.

### What Makes This Special

Existing trading journals (e.g. Stonk Journal) track outcomes. tradingbot tracks decisions — including the reasoning behind them, the overrides against recommendations, and the cumulative cost of emotional behavior. Override tags (N/R/G) with required comments create a structured record that improves with every trading session. The nightly prompt flow is designed to make ignoring feedback feel worse than engaging with it.

Core insight: the problem is not a bad strategy. The problem is the absence of a feedback loop that makes irrational decisions visible and measurable.

## Project Classification

| Attribute | Value |
|-----------|-------|
| Project Type | CLI tool (Python, terminal prompts) |
| Domain | Personal fintech — single user, no commercial compliance requirements |
| Complexity | High — financial data integrity, no-lookahead invariant, strategy version traceability, fill simulation accuracy |
| Project Context | Greenfield |

## Success Criteria

### Go-Live Criteria

- **Primary**: System P&L ≥ weighted performance of referenced sector ETFs, weighted by capital deployed. Measures whether active trading generates enough return to justify the time out of market vs passive ETF hold.
- **Secondary**: R-multiple trend positive and improving over the assessment window. Market-neutral measure of whether the strategy generates positive expected value per trade regardless of market direction.

### Nightly Review Engagement

- **Completion rate** is the primary lead indicator — surfaced in all reports alongside outcome metrics. Measures whether the feedback loop is open at all.
- **Missed-night cost display**: when recovery mode runs, show what happened to open positions during the gap. Makes the cost of absence concrete, not abstract.
- **Pre-recommendation override impact reminder**: if 30d or 90d override impact is negative, display it before the staging phase begins — creates friction at the moment the next override is being considered.

### Override Quality Reporting

- Override impact report shows **p50, p95, p99 distribution** alongside mean — handles outlier skew and makes the feedback loop meaningful earlier (p50 is robust at n=8; mean is not).
- N-tag keyword detection (comment contains "earnings / fomc / macro / fed / cpi / rate / news / guidance" → suggest changing G to N): noted as a future idea, not scheduled for MVP.

### Success Criteria Framing

- **Lag indicators** (outcome): System P&L vs benchmark, R-multiple trend, win rate trend, override impact p50
- **Lead indicator** (behavioral, meaningful earlier): nightly review completion rate

## Anti-Goals

The following are explicit out-of-scope boundaries. tradingbot will never become:

1. **Not a broker integration** — no automatic trade execution, no brokerage API connections. All execution is manual. Autonomy (if ever enabled) is an explicit post-MVP decision, not a default.
2. **Not a screener or signal alert system** — no proactive market scanning or push notifications. Recommendations are generated within the nightly review cycle only.
3. **Not a social or shared tool** — no multi-user support, no cloud sync, no sharing of journals or strategies. Personal and offline, always.
4. **Not a financial advisor** — makes no claim to provide generalizable investment advice. It is a personal decision-tracking tool for Daniel's use only.
5. **Not a data provider** — consumes market data Daniel provides (e.g., OHLCV from a known source); does not own, replace, or scrape market data.
6. **Not a general-purpose portfolio tracker** — tracks trades made within the strategy. Legacy positions (held before the system existed) are out of scope for MVP.

## User Journeys

### Journey 1: The Trader — Nightly Review (Success Path)

It's 9:07pm. Daniel opens a terminal. One command: `tradingbot nightly`.

**Settlement.** The system detected AAPL's stop was breached yesterday — Daniel confirms the fill price, fifteen seconds. Two V-marked entries from last night: MSFT executed at $412.50 (confirmed), TSLA entry at $185.00 never reached (range was $188–$194 — system auto-confirms, Daniel hits enter). Three minutes total.

**Pre-recommendation check.** Before staging loads: *Override impact (last 30d): Gut divergences: -4.2% vs system.* Daniel pauses. He moves on.

**Staging.** Three recommendations appear ranked. Daniel marks V on MSFT and NVDA — he'll submit both to Fidelity tomorrow. The third (GOOGL) he leaves unmarked — auto-logged as not selected, no tag required. System stages MSFT and NVDA.

Eight minutes. Terminal closed. He goes to bed knowing exactly what's set up and why.

**Requirements revealed:** settlement prompt flow, OHLCV-based price reachability check, pre-recommendation override friction, V-mark commitment system, ranked staging display, detail expand, auto-skip for unselected, audit file output.

### Journey 2: The Trader — The Behavioral Pattern (Edge Case)

MSFT has been drifting for three weeks. Daniel has been watching, telling himself it will recover.

**Night 14.** Before tonight's recommendations load:

```
⚠ MSFT — currently $141.20
   Original stop: $155.00 — breached 14 days ago
   Exposure vs. original stop: -$1,820 unrealized
```

The number is concrete now. Not a feeling. Daniel runs `tradingbot position modify --ticker MSFT --stop 138.00`. System records the modification: `modified_stop` divergence, G tag, comment: *expecting recovery on earnings.* System notes current signal is `hold`.

Three sessions later, price hits $138. Exit confirmed at settlement. Next day's override impact report adds a row: *MSFT — stop modified → original $155, exit $138 — adverse impact: $2,340.*

No lecture. Just the number, in the record, accumulating alongside every other G-tag decision. Over time the pattern is undeniable.

**Requirements revealed:** pre-recommendation friction display for positions past exit levels, `position modify` command, `modified_stop` divergence type, unrealized P&L display when price past original exit level, exit-vs-recorded-stop stat in override report.

### Journey 3: The Analyst — Strategy Research

Six weeks of paper trading setup. Before committing real capital, Daniel wants historical evidence.

`tradingbot simulate multi-period --portfolio sim_portfolio.yaml --years 5`

Four window lengths run in parallel. Reports generate. He opens the summary CSV: win rate 58% across all windows, max drawdown 11%, median R-multiple 1.4. He runs it again with a second strategy variant — same periods, same capital. Side-by-side comparison. Version 2 outperforms in 9 of 12 windows, better drawdown recovery.

He explicitly selects rsi_ma_v2 to take forward. The journal records the decision: *paper trading started with rsi_ma_v2 — selected based on simulation results, 9/12 windows outperforming v1.* Not a gut feeling. A documented baseline with a version number attached.

**Requirements revealed:** multi-period simulation, parallel window runs, strategy comparison mode (max 3), per-period and summary CSVs, portfolio file parameter, strategy version selection at phase transition, version recorded at paper trading start.

### Journey 4: The Setup — First Run

Daniel clones the repo on a new machine. The repo includes a template `config/portfolio.yaml` with sensible defaults — sample tickers, 70/25/5 allocations, $50,000 starting capital, strategy version set to `rsi_ma_v1`. It runs without modification. He edits it to reflect his actual tickers and capital, but he doesn't have to before the first run works.

`tradingbot nightly`. Config validates on startup — no missing fields, allocations sum to 100, no unrecognized stop types. Data fetch begins: progress shown per ticker. Four fail history validation — too recently listed. System halts: *NVDA: insufficient history — requires 1,440 trading days, found 980. Use --allow-partial-history to override.* Daniel removes them for now and reruns.

First nightly. No prior journal history — settlement phase is skipped entirely. Three recommendations appear. He approves two, leaves one unmarked. System stages. Audit file written.

Win rate, override impact, P&L comparison: all show "n/a — no closed trades." System states what it is. The record starts here.

**Requirements revealed:** template portfolio.yaml committed to repo, config validation on startup with clear field errors, data fetch progress + history enforcement, `--allow-partial-history` flag, first-run detection (skip settlement), graceful "n/a" state across all stats.

### Journey 5: The Feedback Loop — Live Trading Review

Ninety days in. Daniel runs `tradingbot journal report --period=90d`.

The override impact report opens. Gut divergences: -6.1% average vs system. News divergences: +1.2%. The named insight reads: *Your edge is news, your drag is gut.* He's seen this number grow for six weeks. It's not new anymore — it's a pattern.

He opens the system P&L comparison. The system journal is up 8.4% over the period. His journal: 5.1%. The gap is 3.3 percentage points — two Gut-tagged decisions account for most of it. He knows which ones.

He doesn't change the strategy. He adds a personal note to the journal: *starting to wait on Gut-tag trades — let one pass tonight.* Next week the nightly review will show whether he followed through.

The feedback loop is working. Not because it lectures him. Because the number is there every time he looks.

**Requirements revealed:** periodic report command, override impact breakdown by tag (bar format + named insight), system vs Daniel P&L comparison, period-scoped queries, report filenames with date ranges.

### Journey 6: The Recovery — Missed Nights

Daniel was traveling for five days. `tradingbot nightly` detects the gap and routes to recovery mode. He steps through each missed day in order — settlement and staging run identically to live mode. Twenty minutes total.

After all five days are processed, before tonight's staging loads:

```
Recovery complete — 5 sessions processed.

Missed recommendation cost (reachable during absence, earliest entry per ticker):

  SELLS:
  TSLA  SELL recommended 2026-03-16 @ $185.00 — price today: $171.40  (-$13.60/share if exited)

  BUYS:
  AAPL  BUY  recommended 2026-03-15 @ $182.50 — price today: $189.40  (+$6.90/share if entered)
  MSFT  BUY  recommended 2026-03-16 @ $411.20 — price today: $408.80  (-$2.40/share if entered)

Not a trade record. Earliest reachable recommendation per ticker used.
```

The TSLA number lands. He was holding TSLA. System told him to sell three days ago. He didn't see it. The position is still open, now $13.60/share further underwater.

Tonight's nightly runs as normal. TSLA appears in staging with a fresh sell recommendation.

**Requirements revealed:** post-recovery opportunity cost summary, sells surfaced before buys (higher urgency), price-today vs. recommended-entry delta, "earliest recommendation per ticker" rule, clearly marked as non-trade-record, missed sell feeds directly into tonight's staging if signal persists.

### Journey Requirements Summary

| Journey | Key capabilities |
|---------|----------------|
| 1 — Nightly Review | Settlement flow, price reachability, pre-recommendation friction, ranked staging, V-mark, audit output |
| 2 — Behavioral Pattern | Pre-recommendation stop alert, `position modify`, `modified_stop` divergence, override impact accumulation |
| 3 — Strategy Research | Multi-period simulation, parallel window runs, strategy comparison, version selection at phase transition |
| 4 — First Run | Template portfolio.yaml, config validation, data fetch + history enforcement, first-run detection, graceful n/a states |
| 5 — Feedback Loop | Periodic report, override impact by tag + named insight, system vs Daniel P&L comparison |
| 6 — Recovery | Gap detection, ordered recovery loop, `recovery_mode` flag, missed recommendation cost display (sells first) |
