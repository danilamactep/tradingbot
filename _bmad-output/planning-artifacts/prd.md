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

### Journey 7: The Analyst — Inconclusive Simulation

Daniel has been tuning the strategy for two weeks. He runs `tradingbot simulate multi-period --portfolio sim_portfolio.yaml --years 5`.

Results come back mixed. Win rate 51% — marginal. Max drawdown 18% — high. Median R-multiple 1.1 — thin edge. He runs strategy comparison against version 1: version 2 is slightly better but not decisively — outperforms in 7 of 12 windows, underperforms in 5.

The go-live criteria are surfaced alongside the results: *System P&L ≥ weighted ETF benchmark.* In simulation, the strategy clears the bar in 7 of 12 windows, misses in 5. Marginal.

The system shows numbers. It doesn't recommend. Daniel makes the call: not ready. He records the decision:

`tradingbot journal note "sim results marginal — iterating strategy before paper trading"`

He goes back to strategy configuration. The note sits in the audit trail alongside the simulation report. When he runs the next simulation, the record will show what he saw last time and what he decided.

**Requirements revealed:** go-live criteria surfaced alongside simulation results for self-assessment, `tradingbot journal note` command for lightweight decision recording, phase transition decisions logged in audit trail alongside trade history.

### Journey Requirements Summary

| Journey | Key capabilities |
|---------|----------------|
| 1 — Nightly Review | Settlement flow, price reachability, pre-recommendation friction, ranked staging, V-mark, audit output |
| 2 — Behavioral Pattern | Pre-recommendation stop alert, `position modify`, `modified_stop` divergence, override impact accumulation |
| 3 — Strategy Research | Multi-period simulation, parallel window runs, strategy comparison, version selection at phase transition |
| 4 — First Run | Template portfolio.yaml, config validation, data fetch + history enforcement, first-run detection, graceful n/a states |
| 5 — Feedback Loop | Periodic report, override impact by tag + named insight, system vs Daniel P&L comparison |
| 6 — Recovery | Gap detection, ordered recovery loop, `recovery_mode` flag, missed recommendation cost display (sells first) |
| 7 — Inconclusive Simulation | Go-live criteria surfaced with results, `journal note` for decision recording, phase transition audit trail |
| 8 — Portfolio ETF Stop | Portfolio snapshot in every nightly, ETF stop detection, bond redeployment eligibility, TLT tailwind interpretation, paired VUG + TLT re-entry |

### Journey 8: The Portfolio — ETF Stop and Redeployment

Three months in. Every nightly review opens with the portfolio snapshot:

```
Portfolio — 2026-03-09
VUG   $71,240  (71.2%)  Stop: $68,100  YTD: +4.8%
TLT   $24,890  (24.9%)  Stop: $21,200  since-high: -2.1%
Active trades: 3 positions  Capital at risk: $3,870  (3.9%)
```

Background context. Daniel glances at it and moves to the stock recommendations.

Then one morning the index drops hard. That night, settlement opens differently:

```
⚠ VUG — YTD protection stop breached
  Stop level: $68,100  |  Yesterday low: $67,340
  Execution price?
  > 68100
```

Daniel confirms. The system calculates bond redeployment eligibility immediately:

```
VUG exit recorded.

Bond redeployment check:
  TLT gain since VUG stop date:   +$1,840
  VUG loss at stop:               -$2,960
  Condition not met — TLT has not covered index loss yet.
  Monitoring nightly. You will be prompted when condition is met.
```

No action required. Each subsequent nightly shows the monitoring line before stock recommendations, pulled from the reports module — the same interface that surfaces win rate and override impact:

```
VUG re-entry: watching — TLT coverage: $1,840 / -$2,960 (62%)
TLT: +2.1% (5d) — rates falling → tailwind likely continuing
Simulation: TLT covered VUG loss in 8 of 10 similar events (avg 14 days)
```

The third line is what makes holding a reasoned position rather than hope. It comes from the bond hedge effectiveness history in the reports module — simulation-based until live stop events accumulate, then derived from actual journal history.

Twelve days later, TLT has gained enough. Tonight's staging opens with a portfolio-level event before stock recommendations:

```
⚠ PORTFOLIO — Bond redeployment condition met
  TLT gain: +$3,100 | VUG loss: -$2,960 — threshold exceeded.

  Paired re-entry recommendation:
  VUG  BUY to 70% target allocation  |  Entry: market open
  TLT  RE-ENTER to 25% target allocation  |  Entry: market open

  Approve both / review individually / skip
```

Daniel approves both. System stages them. The portfolio snapshot on the next nightly reflects the restored structure.

**Requirements revealed:** portfolio snapshot at top of every nightly (all three buckets, stop levels, allocation %); ETF stop detection at settlement using trailing stop logic distinct from stock ATR stops; bond redeployment eligibility check on VUG exit; nightly monitoring display (TLT coverage %, TLT momentum with tailwind/headwind interpretation derived from ^TNX direction, simulation-based bond effectiveness stat); all monitoring stats fetched from reports module consistent with JournalReader interface; reports module bond hedge effectiveness query transitions from simulation-based to journal-based as live history accumulates; paired VUG + TLT re-entry as portfolio-level event before stock recommendations; approve-both shortcut for paired re-entry.

### Unresolved Edge Cases — Carry Forward to Functional Requirements (Step 7)

**Journey 1 — Nightly Review**
- Fill price entered outside day's OHLCV range: validate fill within [low - tolerance, high + tolerance]; prompt re-entry or flag as gap fill. Risk: invalid fill corrupts P&L and all downstream reports.
- Daniel modifies a recommendation during staging (entry/stop/target): capture modified fields as `independent_buy` divergence; require tag + comment on deliberate changes. Risk: modification not recorded; override impact report has no record.
- Zero recommendations generated tonight: show explicit 'No recommendations tonight — [reason]' message; skip staging phase cleanly. Risk: Daniel unclear if system failed or strategy found no setups.
- Capital-at-risk cap fully consumed before staging: show recommendations as visible but non-stageable with cap-hit notice; suggest trim candidate. Risk: Daniel sees recommendations he can't act on with no explanation.

**Journey 2 — Behavioral Pattern**
- Daniel modifies stop multiple times on same position: each modification creates a new `modified_stop` divergence record with timestamp. Risk: only latest stop tracked; override impact report understates pattern.
- System has active SELL recommendation AND position has modified stop — same ticker, same night: surface SELL recommendation first; treat acceptance as exit; `modified_stop` divergence only if Daniel overrides sell. Risk: ambiguous prompt ordering with no defined resolution.

**Journey 3 — Strategy Research**
- Ticker delisted or data gap mid-simulation window: detect missing trading days above threshold; halt with clear error identifying ticker and gap dates. Risk: simulation silently skips days; metrics computed on incomplete data.

**Journey 4 — First Run**
- yfinance unavailable (no internet, rate limited, API change): catch fetch failure per ticker; halt with actionable error distinguishing network vs rate limit vs API failure. Risk: silent partial fetch; simulation runs on incomplete history.
- All tickers fail history validation — no instruments remain: after removing failing tickers, check instruments list is non-empty before proceeding; halt with clear message. Risk: system proceeds with empty portfolio; no recommendations with no explanation.
- Strategy version referenced in portfolio.yaml does not exist on disk: validate strategy version file path at startup; halt with 'strategy config not found: rsi_ma_v1.yaml'. Risk: system fails at recommendation time, not startup; no audit trail.

**Journey 5 — Feedback Loop**
- Zero divergences in requested period: show 'No divergences recorded in this period'; suppress named insight. Risk: named insight generates from empty data; misleading output.
- Only one tag type used across all divergences (e.g., all G, no N or R): show only tags with data; suppress comparison language when fewer than 2 tag types present. Risk: named insight compares G against nothing; misleading framing.
- System journal has no closed trades but Daniel's does (or vice versa): clearly label the other as 'insufficient history'; suppress P&L comparison until both journals have data. Risk: P&L comparison computed against zero baseline; appears as 100% outperformance or underperformance.

**Journey 6 — Recovery**
- Daniel closes terminal mid-recovery (e.g., after 3 of 5 missed days): persist recovery progress to journal with `last_recovery_date`; resume from next unprocessed day on next run. Risk: recovery restarts from day 1; duplicate journal entries.
- Opportunity cost calculation run before today's price data fetched: run data fetch before opportunity cost display; show 'price unavailable' if fetch fails rather than stale or zero price. Risk: opportunity cost displayed with wrong prices.
- Same ticker had BUY recommendation on day 1 and SELL on day 3 of missed period: surface sell under SELLS section independently; do not suppress sell because earlier buy exists for same ticker. Risk: sell missed opportunity hidden; cost of holding through missed sell signal not shown.
- Gap spans market holidays or weekends (5 calendar days = 2 trading days): use `pandas_market_calendars` to count missed trading days, not calendar days; recovery loop iterates trading days only. Risk: recovery prompts for non-trading days; settlement runs on days with no OHLCV data.

**Adversarial Review — Carry Forward to Functional Requirements (Step 7)**
- V-mark undefined (Journey 1): define what V-mark is — keypress, menu selection, or flag — and specify its behavior (commit, undo, bulk skip). Risk: ambiguous UI term in multiple journeys with no implementation signal.
- Manual Fidelity submission invisible (all journeys): define the handoff between "system staged" and "order submitted to broker" — what does Daniel see, what can go wrong, how does the system handle a staged order that was never submitted? Risk: most failure-prone moment in the workflow has zero requirements coverage.
- `position modify` not in CLI (Journey 2): define command signature, modifiable fields (stop, target, size), validation rules, and which fields require a divergence tag. Risk: journey introduces an MVP command not in the CLI conventions table.
- Strategy selection recording (Journey 3): `tradingbot journal note` (introduced in Journey 7) resolves the mechanism; update Journey 3 to reference it explicitly and define what note text is required at phase transition.
- Recovery skip constraint rationale (Journey 6): "Daniel cannot skip ahead" is a significant UX constraint presented without justification. Define the rationale (data integrity, audit trail completeness) and whether any shortcut is acceptable (e.g., bulk-confirm "no action taken" for all missed days). Risk: highest adoption friction point in the product with no stated design rationale.
- Behavioral follow-through tracking (Journey 5): "next week the nightly review will show whether he followed through" implies a mechanism that does not exist. Either define how the system tracks behavioral change after an override impact insight, or remove the implied capability. Risk: sets an expectation the system cannot meet.
- Exit confirmation language (Journey 2): "exit confirmed at settlement" is ambiguous — clarify that the system detects the stop breach via OHLCV and Daniel confirms the fill price; system never auto-books an exit. Risk: blurs the system-detects / Daniel-confirms distinction that is a core data integrity invariant.
- Template config fragility (Journey 4): "runs without modification" depends on template tickers having sufficient yfinance history. Document which tickers are in the template and confirm they meet the history requirement, or add a note that the template may require ticker substitution on first run. Risk: first-run promise breaks silently if a template ticker has been delisted or lacks history.
- Personal note mechanism (Journey 5): Journey 5 says "he adds a personal note to the journal" without a command. Resolve by updating Journey 5 to reference `tradingbot journal note` from Journey 7 for consistency.
