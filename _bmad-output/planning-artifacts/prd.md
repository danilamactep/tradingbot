---
stepsCompleted: [step-01-init, step-02-discovery, step-02b-vision, step-02c-executive-summary, step-03-success-elicitation-in-progress]
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

## Success Criteria — Elicitation Findings

> Pre-mortem completed. Anti-Goals in progress. This section will be finalized once Anti-Goals is complete.

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
