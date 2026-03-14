---
stepsCompleted: [1, 2]
inputDocuments: []
date: 2026-03-07
author: Daniel
---

# Product Brief: tradingbot

---

## 1. Problem Statement

Daniel is an intermediate developer-trader who has been trading for several years but losing money. The root cause (via 5 Whys analysis) is not a bad strategy — it is the **absence of evidence to reason from**. Without accumulated proof that the system's judgment is reliable, decisions default to intuition rather than data. Overrides fill the gap left by absent confidence. The system's purpose is to automate the collection of that evidence and make every decision a reasoned one.

**North star: Evidence over intuition — reasoned decisions, earned automation.**

> It's 9pm. You open the terminal, type `tradingbot nightly`, and the system picks up where yesterday left off.
>
> First, settlement. It detected that AAPL's stop level was breached at the open. You confirm the execution price — fifteen seconds. It asks about the two staged orders from last night. You enter the fills. The journal updates. That part takes three minutes total.
>
> Then tonight's recommendations appear. Four of them — stocks and ETFs, ranked by opportunity. Each one fits on a single screen: entry, stop, target, risk/reward, the signal that triggered it. You open TradingView on the second monitor and confirm the setup looks right. Two you approve immediately. One you skip — your choice, not a lack of options. The fourth you modify slightly. The system stages everything for tomorrow.
>
> Eight minutes, start to finish. You close the terminal and go to bed knowing exactly what's set up and why.
>
> Six months in, the picture is clear. Your portfolio has three layers working in parallel: VUG compounding steadily in the background, TLT providing a cushion when markets get rough, and a focused active sleeve where the real learning happens. Each layer has rules. When VUG stops out, you know what to watch before re-entering. When bond gains cover the index loss, the system tells you — thesis working. When they don't, it flags it — regime to understand.
>
> Your active sleeve never runs dry. The system always has suggestions. On slow weeks there are fewer; on volatile weeks there are more. You're never sitting idle wondering what to do with available capital.
>
> The performance picture builds over time without you assembling it. Win rate by signal type. P&L delta between what you did and what the system recommended. Which override tags are costing you and which are earning. The system doesn't lecture — it just shows the numbers, every session, quietly accumulating into a picture you couldn't have constructed manually.
>
> When you want to test a different stock list or a new strategy, you spin up a simulation portfolio and replay two years of history in minutes. Reports come out the other side: win rate, drawdown, capture ratio, period-by-period. You compare versions side by side and make the call with evidence.
>
> The routine is calm. Not exciting. That's the point. The decisions are reasoned, the records are clean, and the portfolio is growing. You know exactly why every trade happened, and the system is slowly, measurably, earning the right to do more.

### Why Not an Existing Tool?

Stonk Journal (app.stonkjournal.com) was explicitly evaluated and rejected. It lacks:
- Stop loss as a first-class field
- Override tracking with reason tags
- System-recommended vs actual P&L comparison
- Strategy version linking per trade
- Nightly conversation logs

A custom system is necessary.

---

## 2. Product Vision

A personal, offline Python toolkit that:

1. Recommends trades based on a coded, versioned strategy
2. Tracks every decision Daniel makes — including overrides
3. Compares actual P&L vs what the system would have produced
4. Generates nightly reviews that Daniel must actively engage with
5. Replays strategies and portfolios across years of market history at speed, generating reports detailed enough to compare, iterate, and decide with confidence
6. Earns enough trust, through demonstrated performance, that following its recommendations becomes the default and overriding them becomes the exception

The system is not a black box. Daniel retains full control. The system's job is to accumulate evidence of its own reliability — so that over time, trusting it becomes the reasoned choice.

---

## 3. Core Philosophy

| Principle | Detail |
|-----------|--------|
| Evidence first | The journal and override tracker exist to build a proof record — not to catch bad behavior, but to answer: does the system's judgment outperform mine yet? |
| Minimal friction, maximum clarity | Nightly review must be fast; every session should leave Daniel with clearer evidence to reason from |
| Boring technology | Python, SQLite, file I/O — no cloud, no streaming; `multiprocessing` allowed for parallel simulation runs |
| Audit trail everything | Every recommendation includes a full JSON reasoning snapshot |

---

## 4. MVP Scope

### MVP Components

| Component | Notes |
|-----------|-------|
| **Journal** | SQLite + Alembic, trade recording, override tracking |
| **Strategy Engine** | Python plugin class + versioned YAML config; metrics calculated internally |
| **Data Fetcher** | yfinance automation included — manual CSV management is too painful |
| **Portfolio Manager** | Required for position sizing, capital-at-risk tracking, trim logic |
| **Nightly Review** | Terminal prompts, recommendation display, override capture, audit file output |
| **Replay Mode** | Fill simulation, automated replay across historical data |
| **Rebalancing Calculator** | MVP required — backtest simulation must model quarterly rebalancing events to produce accurate portfolio progression; prices fetched automatically via Data Fetcher |

### MVP scope rationale

The Rebalancing Calculator was nearly deferred but is required for MVP because: without modeling quarterly rebalancing in multi-period simulation, portfolio progression numbers are inaccurate (static 70/25/5 allocation is unrealistic). Re-running simulations after adding it post-MVP would invalidate earlier decisions. The simulation logic and operational report (`--preview`) share enough code that splitting them adds complexity without benefit.

### Post-MVP (explicitly deferred)

- **LLM-assisted strategy research** — use an LLM (Claude or similar) to analyze replay and simulation reports alongside strategy definitions to suggest improvements: identifying parameter ranges worth testing, spotting patterns in losing periods, proposing signal combinations not yet tried. This is a strategy research assistant, not a report summarizer — the value is in surfacing what the data implies about strategy design. Requires substantial replay report history before it's useful. Prompt design, data scoping, and evaluation criteria to be defined before implementation.
- **Cancelled order tracking** — store unfilled/cancelled staged orders to help calibrate strategy entry thresholds (e.g., entry limits frequently missed by small margins suggest threshold needs tightening).
- **Confidence signal (red/yellow/green)** — visual confidence rating per recommendation abstracted from the ranking score. Enables faster scanning without computing R/R and win rate mentally. Categorization criteria need further discussion before implementation.
- **Pre-nightly checklist** — short prompt before recommendation review: macro conditions, earnings this week for held positions, VIX level. Prompted by system, answered by Daniel. Checklist items to be defined.
- **Add / reduce position actions** — `add` and `reduce` as first-class strategy actions alongside `buy / sell / hold / no-action`. Requires defining skip logging behaviour for each.
- **"Ask for more" recommendations** — option to request additional recommendations beyond default display in nightly staging phase.
- **Missed opportunity tracking** — store non-acted "buy" signals for tickers defined in `config/portfolio.yaml` instruments list to enable a report showing what the system would have earned on signals Daniel never took. Bounded scope prevents noise.
- **Execution deviation tracking** — capture slippage (set price vs actual Fidelity fill price) per trade. Schema: `set_entry`, `actual_entry`, `set_stop`, `actual_stop`, `set_target`, `actual_target`. Deviation computed at report time. Deferred due to nightly review friction cost; trivial to add via Alembic migration.
- **Market regime labels** — tag fetched data with bull/bear/sideways market state and rate direction (rising/falling). Enables strategy performance analysis by regime.
- **Pre-market entry modification tracking** — "modify" action in nightly review should record which field was changed (entry/stop/target/size). Enables Daniel to see pre-market adjustment habit reflected in data over time.
- **Legacy position management** — positions held before the system was set up have no entry recommendation, no override history from inception, and break the feedback loop. Excluded from MVP. Post-MVP: define onboarding flow for pre-existing positions (manual journal entry with no system recommendation baseline).
- **Broker automation** — automate outcome capture via broker API integration, replacing manual settlement prompts.
- Scheduled runner (cron wrapper around nightly script — deployment concern, not a component)

### No lookahead bias (invariant across all modes)

No component has knowledge of future prices — ever. In replay and multi-period simulation modes, the system feeds OHLCV data one day at a time, exactly as it would in live mode. `MarketSnapshot.ohlcv` contains only up to `min_history_days` trading days (default 220) up to and including `as_of_date`. The day's close price is never available until settlement the following day. This invariant must be enforced at the data feed layer, not assumed by individual components.

**Regression test required**: an automated test must assert `max(ohlcv.index) <= as_of_date` across all code paths that construct `MarketSnapshot`. A design invariant without a test is not an invariant.

---

## 5. Components

### 5.1 Data Fetcher

- **Library**: `yfinance` with `auto_adjust=True` (handles splits/dividends automatically)
- **Storage**: gzipped CSVs (`historical_prices/AAPL.csv.gz`) — append-only, excluded from git
- **Re-fetch trigger**: `ticker.actions` checked for new corporate events since last fetch. When a corporate event is detected: re-fetch all data from the earliest date available locally in the gzip, capped at `max_history_years` (from config). Do not fetch beyond this cap regardless of local history.
- **Corporate event validation**: after re-fetch, run a price continuity sanity check — if the ratio between consecutive closing prices at the event date exceeds a threshold inconsistent with normal volatility (i.e., looks like an unadjusted split), treat as a hard failure. Bad price data corrupts all downstream metrics; failing loudly is safer than silently continuing with corrupted ATR/RSI values. Threshold to be defined during implementation.
- **Pluggable design**: yfinance sits behind a fetcher interface as an architectural guardrail. The interface is not expected to change or be reimplemented — this is a protective boundary, not an active extensibility plan.
- **Validation**: Total data loaded per ticker = `max_history_years` (from config, default 5) × trading days + `min_history_days` (from strategy YAML, default 220). The extra 220 days acts as the indicator warm-up period before the simulation window begins. Uses `pandas_market_calendars` for calendar validation.
- **New ticker onboarding**: Full history requirement (`max_history_years + min_history_days`) must be satisfied before a ticker is used in simulation. If a ticker lacks sufficient history (e.g., recently listed), this is a hard failure by default. Override with `--allow-partial-history` CLI flag — in this mode, the fetcher uses whatever history is available and the simulation window is shortened accordingly. No partial evaluation without explicit override.
- **Scope**: fetches all instruments + their reference indices defined in `config/portfolio.yaml` + ^TNX (10-year Treasury yield, used for TLT re-entry rate direction signal)

### 5.2 Strategy Engine

**Four-layer architecture:**

| Layer | Responsibility | Testable against |
|-------|---------------|-----------------|
| `Metric` | Raw computed number from OHLCV. No interpretation. | TradingView golden inputs |
| `Indicator` | Evaluates metric(s) against threshold → boolean/state. Thresholds live in YAML config. | Known metric values |
| `Signal` | Composes indicators → trade action (buy/sell/hold/no-action). | Known indicator states |
| `Rule` | Filters/modifies signal using journal history + market context. MVP scope: only rules with explicit, measurable thresholds (e.g., VIX level, capital-at-risk cap). Regime-based rules (bull/bear/sideways, macro conditions) are post-MVP. | Known journal/market context |
| `Strategy` | Thin orchestrator — wires metrics → indicators → signals → rules → `Recommendation`. | End-to-end |

- **Layer 1**: Python classes implementing each layer
- **Layer 2**: Versioned YAML parameter config file (thresholds, weights, periods) — used by Indicators
- Config version is linked to each trade for full backtest traceability
- **Config hash enforcement**: YAML config is hashed at recommendation time; hash stored alongside version string. On load, hash is recomputed — mismatch is a hard failure.

_Core class signatures (`Metric`, `Indicator`, `Signal`, `Rule`, `RuleContext`, `Strategy`) to be fully specified in the architecture doc._

**Strategy dry run mode (MVP priority — not a dev utility):**
- Feed one day's OHLCV, see the full recommendation output
- Used to test a new strategy config without running a full replay
- `tradingbot strategy dry-run --ticker AAPL --date 2026-03-09`
- **Output parity**: dry-run and nightly review staging phase share the same output formatting function — divergence is architecturally impossible, not just documented.

**Position sizing (ATR-based, 2% portfolio risk rule):**
```
shares = (portfolio_value × 0.02) / (entry - stop)
```
- `portfolio_value` = cash balance + cost of open positions (not mark-to-market) — used for sizing only
- `unrealized_pnl` = mark-to-market value of open positions minus cost basis — visible in nightly reports and portfolio snapshot for awareness; never used for sizing or cap calculations
- When capital-at-risk cap is reached: trim existing position first (ranked by target proximity → momentum → hold time), then size down new trade to remaining headroom

### 5.3 Data Model

_Key dataclasses (`MarketContext`, `MarketSnapshot`, `Recommendation`, `Position`) to be fully specified in the architecture doc. Key fields: `MarketSnapshot` carries 220 days of OHLCV + reference index data + portfolio context. `Recommendation` carries action, entry/stop/target, R/R, momentum, relative strength, rule warnings, persistence flag, and a full reasoning dict for audit trail._

**Unified portfolio config (`config/portfolio.yaml`):**

Tickers and portfolio allocations are unified in a single file — `tickers.yaml` is removed. All instrument definitions live in `portfolio.yaml`. Simulation can use a separate portfolio file (`--portfolio sim_portfolio.yaml`) to test different compositions without touching the live config. See `docs/data-model-sketch.md` for the full YAML structure.

**Instrument type derivation** — `type` field is not stored; behavior is derived at runtime:
- `reference_index` present → stock: ATR-based sizing, fixed stop, 5-day re-entry cooldown
- `stop: ytd_protection` → index ETF: allocation-based sizing, YTD gain protection formula
- `stop: trailing_15pct` → bond ETF: allocation-based sizing, 15% trailing stop

**Portfolio state derivation** — constructed at the start of each nightly run from journal history. Never stored redundantly:

```
open positions  = all buys without a corresponding exit
cash balance    = starting_capital - Σ(entry costs) + Σ(exit proceeds)
capital_at_risk = Σ(cost basis of open stock positions)
risk_basis      = cash + capital_at_risk         ← used for position sizing and cap enforcement
market_value    = risk_basis + unrealized_pnl    ← what the account is actually worth today
```

Both values are displayed in reports and the nightly portfolio snapshot, clearly labeled. `risk_basis` is a non-standard definition — reports include a brief note explaining that sizing uses cost basis, not market value, to prevent misreading the number as account value.

**Regression test required**: golden input set of known trade history → expected positions, cash balance, capital-at-risk, and portfolio value. This derivation is the foundation of position sizing and cap enforcement — a silent error here corrupts everything downstream.

### 5.4 Journal (SQLite + Alembic)

**Purpose**: The journal exists to answer two related questions: (1) does the system's autonomous judgment outperform Daniel's discretionary trading? (2) on the trades Daniel did make, did his judgment add or destroy value versus what the system recommended? It does this by maintaining two fully independent trade records and comparing them over time.

**Two journals, one database**: Both journals live in the same SQLite database using a single `trades` table with a `portfolio_id` column. A `portfolios` table defines each journal (`name`, `starting_capital`, `created_at`). Adding a new journal type requires one INSERT into `portfolios`, no schema migration, no code change. `JournalReader` takes a `portfolio_id` parameter and works identically for any journal.

**Shared infrastructure**: Both journals use identical position sizing, stop/target logic, capital-at-risk cap, and starting capital. The only difference between journal types is the **picker** — the logic that decides which ticker to act on each night. Portfolio state is derived independently per `portfolio_id` at the start of each nightly run.

**System journal — fully autonomous**: The system journal always executes the top-ranked recommendation each night. It never waits for Daniel, never mirrors Daniel's decisions, and never skips because Daniel skipped. It independently manages its own positions from entry to exit based solely on system signals. Both journals may hold positions in the same ticker at the same time — they manage them independently.

**Post-MVP journal types**: additional pickers using the same shared infrastructure — second pick, random pick (baseline benchmark). One INSERT into `portfolios` per new journal type, no code change.

**Hold recommendations stored**: every nightly recommendation for a currently held position (VUG, TLT, active trade) is journaled, including "hold." This enables bond hedge effectiveness queries (e.g., "did the system recommend selling TLT on the day VUG stopped?"). Unpositioned tickers with "no-action" are NOT stored — noise with no interpretive value.

**Technical:**
- **Read pattern**: read-all → compute → write-once (no concurrent access, no WAL mode needed)
- **All writes transactional** — every journal write wrapped in an explicit SQLite transaction. On startup, run `PRAGMA integrity_check` and halt if journal is inconsistent.
- Migrations managed by Alembic
**Trade reasoning — symmetric for both journals:**
- **System reasoning snapshot** (stored per system trade): the full indicator context at recommendation time — ticker, source file + hash, date range, RSI, MAs, ATR, VIX, SPY return, capital-at-risk pct, rules warned/blocked. Answers: *why did the system act?*
- **Human reasoning record** (stored per divergence): N/R/G tag + comment. Answers: *why did Daniel deviate?*

Both are reasoning records attached to trade decisions. Format differs; concept is identical. Stored in a `reasoning` table linked to trades, with a `source` field (`system` | `human`) and a JSON `payload`. See `docs/data-model-sketch.md`.

- Git tagging for backtest traceability: `git tag backtest-AAPL-2026-03-09`

**JournalReader** (read-only wrapper) — methods: `win_rate`, `consecutive_losses`, `signal_win_rate`, `divergence_rate`, `divergence_quality_score`, `divergence_pnl_by_tag`. See `docs/data-model-sketch.md` for full interface.

### 5.5 Divergence Tracking

A divergence occurs whenever Daniel's action differs from the system's recommendation. At settlement, the system detects divergences automatically by comparing Daniel's journal entries against that night's recommendations — Daniel never needs to manually flag a deviation.

**Four divergence types:**

| Type | What happened | Reason required |
|------|--------------|----------------|
| `ignored_buy` | System recommended buy; Daniel didn't act | Yes |
| `ignored_sell` | System recommended exit; Daniel held | Yes |
| `independent_buy` | Daniel's entry price or stop differed from recommended | Conditional (see below) |
| `independent_sell` | Daniel exited; system had no sell signal | Yes |

**Entry price vs. stop divergence handling:**
- **Entry price difference**: at settlement, prompt — "Fill difference or deliberate price change?" If deliberate: reason tag + comment required. If fill/slippage: no reason required, recorded as execution noise. Fallback: threshold-based auto-classification if prompting proves too disruptive.
- **Stop difference**: always deliberate — reason tag + comment always required. System auto-detects stop divergences from the trade record vs. recommendation.

Note: the system journal always executes top pick autonomously — an ignored buy is a divergence between Daniel and the recommendation, not between the two journals. The journals are a performance benchmark; divergence tracking is a decision-quality tracker. These are separate comparisons.

**Divergence tags** (one required on deliberate divergences) + **comment** (always required):
- `N` — News/Event (any external catalyst: macro, earnings, sector move)
- `R` — Risk (portfolio-level concern: position size, drawdown, capital at risk)
- `G` — Gut (intuition or any reason that doesn't fit N or R)

**Reporting:**
- P&L delta (actual vs system) derived at report time by comparing both journals — no manual tracking required
- Tag performance breakdown: P&L delta per tag (N/R/G). Terminal bar format: `G ████████ 60% (-6.1%)` `N ███ 20% (+1.2%)`
- Divergence quality score: % of divergences where Daniel outperformed the system, tracked over rolling 30d and 90d windows
- Named insight: when tag performance is asymmetric, surface plain-language summary (e.g. "News divergences: +1.2% avg | Gut divergences: -6.1% avg — your edge is news, your drag is gut")
- Win rate and divergence stats surfaced from first closed position / first divergence respectively. No minimum floor; "n/a — insufficient history" shown only when zero records exist.

### 5.6 Nightly Review (Terminal Prompts)

The nightly command has two sequential phases in live mode:

1. **Settlement phase**: Daniel reports execution outcomes via terminal prompts at the start of each nightly run. The system generates the recommendations and is the source of truth for all recommended prices. Daniel is the source of truth for execution outcomes. The system never infers execution status from OHLCV in live mode.

   **Settlement prompt flow (live mode):**

   For each recommendation from the previous night:
   ```
   AAPL was recommended yesterday. Execution price? (price / skipped / [enter] = not submitted)
   > skipped
   Reason: N=News/Event  R=Risk  G=Gut
   > G
   Comment: felt overextended given macro
   ```
   - `execution_price` entered → **executed** → logged as open or closed position
   - `skipped` → logged as skip with reason + comment (applies to SELL on open positions; BUY skips are post-MVP)
   - `[enter]` / not submitted → ignored; no journal entry (BUY recommendations only)
   - Orders are one-day-only — no carry-forward

   After recommendation outcomes, system checks OHLCV for all open positions and surfaces detected limit triggers:
   ```
   ⚠ AAPL — stop level $168.00 breached yesterday (low: $164.20)
   Execution price?
   > 168.00
   Comment (optional): gap down overnight
   ```
   - System detects potential triggers from OHLCV (price crossed stop or target level) and prompts Daniel to confirm with actual execution price
   - Daniel is the source of truth on execution price — system only detects, never assumes
   - System also asks: `Any other triggers not listed above? (y/n)` to catch edge cases the OHLCV detection missed

   System writes all outcomes to an internal CSV as the audit record. Daniel never edits the CSV directly.

2. **Staging phase**: present tonight's recommendations for approval, stage approved orders for tomorrow. Recommendations are ranked by `risk_reward × historical_win_rate` — top opportunities shown first. If yesterday's order expired and conditions persist, the strategy engine recommends again independently — surfaced via the `is_persistent` flag.

   **Recommendation ranking formula (MVP):**
   ```
   score = risk_reward × historical_win_rate(signal)
   ```
   - `historical_win_rate` sourced from `JournalReader.signal_win_rate(signal_name, last_n)` — defaults to 0.5 when insufficient history
   - Two alternative scores computed alongside but not used for ranking (MVP):
     - **EV**: `(win_rate × avg_win) - (loss_rate × avg_loss)` — accounts for magnitude
     - **Conviction**: `R/R × signal_strength × (1 - capital_at_risk_pct)` — incorporates portfolio state
   - All three scores surfaced in periodic reports for formula comparison over time — ranking formula switches post-MVP once data shows which predictor is strongest

Paper and replay modes skip the CSV input entirely — fills are simulated via fill simulation rules (section 5.9). Override tracking applies identically in paper mode as in live mode.

**Missed night recovery:**
If Daniel misses one or more nights, the system detects the gap by comparing today's date to the last journal entry date. On next run, it steps through each missed day in order (oldest first) before running tonight's review:
- **Settlement phase**: runs normally per missed day using that day's OHLCV for stop detection
- **Staging phase**: shows what the system would have recommended that night; Daniel reports whether anything was acted on
- All missed-day sessions are marked `recovery_mode: true` in the journal for auditability
- After all missed days are processed, tonight's nightly review runs as normal

**Interactive replay mechanics:**
- **Date stepping**: auto-advances after each staging phase — no manual `[n]ext` prompt required
- **Starting capital**: read from `starting_capital` in the portfolio file passed via `--portfolio`. Each simulation config defines its own capital.
- **Strategy config**: current config version only (MVP) — no historical config snapshot support
- **Ambiguous exit prompt**: fires identically to live mode when open is between stop and target on the same day both limits are hit

**Execution note**: The system generates recommendations and stop levels. Daniel executes all orders manually in Fidelity (MVP). Outcomes are captured via the settlement prompt flow at the start of each nightly run.

**Terminal prompt format — tiered display:**

Action line is always shown first. Signals follow. Context is available on demand via `[d]` to keep the prompt scannable.

```
# Nightly Review — 2026-03-09

## AAPL — BUY RECOMMENDED ★ PERSISTENT (3 days)
Entry: $182.50 | Stop: $168.00 | Target: $210.00 | R/R: 3.2:1
Override impact (last 30 days): -4.2% vs system

Action: approve / skip / modify / [d]etail
```

Selecting `[d]` expands full context inline:

```
## AAPL — detail
Stop distance: 7.9% | Momentum 5d: +2.3% | Momentum 20d: +8.1%
Reference (QQQ) 5d: +1.2% | Relative strength: +1.1%
RSI: 28 (oversold) — Price at 50MA support ($181.20)

Action: approve / skip / modify

[If skip selected on SELL recommendation:]
Reason: N=News/Event  R=Risk  G=Gut
Comment: ___
```

**"Both stopped" display** — portfolio-level event, shown before per-ticker section:

```
# Nightly Review — 2026-03-09

⚠ PORTFOLIO EVENT — VUG + TLT BOTH STOPPED OUT
This is a portfolio-level event. Per-ticker stops are suppressed.
Waiting for VUG re-entry signal. TLT will re-enter paired with VUG.

[d] View re-entry context
```

**Audit file output**: The nightly conversation (prompts + Daniel's responses) is saved to a dated file as audit trail output. It is not an input file — Daniel does not edit it.

**Ambiguous exit (same day stop AND target hit):** Resolved in two stages:

1. **Deterministic resolution** (using OHLCV only — no intraday fetch):
   - `open > target` → gap up → target hit first
   - `open < stop` → gap down → stop hit first
   - These cases are already covered by fill simulation rules (section 5.9)

2. **Truly ambiguous** (open is between stop and target, both hit same day): prompt Daniel for manual entry. No journal entry written until resolved. Resolution stored with `manual_resolution: true` flag — queryable for future analysis. Expected to be rare.

**Rebalancing notification** (non-blocking): When the quarterly rebalancing report auto-runs, the nightly prompt includes: `"Quarterly rebalancing report generated — review when convenient."` followed by the file path. Does not block trade approvals.

### 5.7 Position Scorecard (Trim Decisions)

Fields: `ticker`, `unrealized_pnl_pct`, `distance_to_target_pct`, `days_held`, `momentum_5d`, `momentum_20d`, `reference_index_return`, `relative_strength`. See `docs/data-model-sketch.md` for full `PositionScore` dataclass.

### 5.8 Autonomy Progression (Four Phases)

| Phase | Mode | Human role | Position sizing |
|-------|------|-----------|----------------|
| 1. Interactive replay | Replay | Decides at each step | N/A (simulated) |
| 2. Paper trading | Paper | Decides at each step | N/A (simulated) |
| 3. Early live | Live | Decides; no auto-execute | Reduced — prove behavior holds with real money |
| 4. Proven live | Live | Optional; `auto_execute` available | Full — go-live criteria met |

**Key distinction**: Automated replay tests the strategy. Paper trading tests strategy + human behavior. These answer different questions.

| Mode | Purpose | Human decisions |
|------|---------|----------------|
| **Automated replay** | Strategy validation across historical periods | None — fully automated |
| **Interactive replay** | Workflow practice, building intuition | Daniel decides at each step |
| **Paper trading** | Tests strategy + human behavior combined | Daniel decides at each step |
| **Live (early)** | Proves real-money behavior with reduced risk | Daniel decides |
| **Live (proven)** | Full execution with optional `auto_execute` | Daniel or system |

### 5.9 Fill Simulation Rules (Paper/Replay)

| Scenario | Fill Price | Gap Flag |
|----------|-----------|----------|
| Normal stop hit (open > stop, trades down to stop) | Stop price | false |
| Gap down past stop (open < stop) | `(min(stop, open) + low) / 2` | true |
| Normal target hit (open < target, trades up to target) | Target price | false |
| Gap up past target (open > target) | Target price | false |
| **Both hit, gap up** (open > target, stop also breached) | Target price (hit first) | false |
| **Both hit, gap down** (open < stop, target also hit) | Gap fill price (stop hit first) | true |
| **Both hit, no gap** (open between stop and target) | Manual entry required | — |

**Gap exit journal fields**: For gap-down exits, journal records all four values: stop price, open, day low, simulated fill, and gap flag (true/false). For manual resolutions, records stop price, target price, open, and `manual_resolution: true`.

### 5.10 Go-Live Criteria

Moving from one phase to the next (replay → paper → live) requires ALL conditions met:

1. System P&L ≥ +10% over trailing 3 months of simulation
2. System P&L > Daniel's actual P&L over same period
3. No recent deterioration: last 2-week win rate ≥ 50% of 3-month average win rate. When this flags, cross-reference simulation results to contextualize
4. Simulation across `max_history_years` at multiple window lengths reviewed and understood by Daniel

**Stopping rules (suspend live trading):**
- 15% drawdown from peak
- 5 consecutive losses

### 5.11 Strategy Simulation & Performance Reports

**Purpose**: Run the strategy across years of historical data at configurable window lengths to understand its behavior before committing real capital, and to track ongoing performance after going live. The decision to move to paper trading is made based on simulation results — there is no separate calibration step.

**Simulation engine:**
- Replay across last `max_history_years` (default 5) of data
- Configurable rolling window lengths: 6, 12, 18, 24 months — run all by default, or specify via `--window`
- Includes portfolio-level positions: VUG and TLT simulated in parallel with active trades
- Run on quarterly cadence (scheduled) + manual command + on-demand before any phase transition (e.g., moving to paper trading)

**Performance metrics (captured per window, summarised across windows):**
- Total trades, average trades per month, trade frequency (trades per week/month by market condition)
- Time in trades vs in cash — % of period with capital deployed
- Win rate
- Quintiles of trade P&L, average win, average loss, best/worst trade
- Longest winning streak, longest losing streak, average streak length
- Quintiles of holding period, average hold, shortest/longest
- Average risk/reward ratio, average R-multiple per trade
- Max drawdown, average drawdown, recovery time from max drawdown
- Percentage stopped out, percentage hit target, percentage manual close
- Expected performance range: median, best, worst across all windows — surfaces what "normal" looks like for this strategy

**Output — two-section report:**
- **Section 1**: Per-period detail CSV — all metrics for each window
- **Section 2**: Summary statistics CSV — median, best, worst for every metric across all windows

**Bond hedge effectiveness tracking** (per index-stop event):

| Category | Condition | Attention level |
|----------|-----------|----------------|
| **Triggered** | `bond_gain >= index_loss` | Normal — thesis worked |
| **Near miss** | `bond_gain >= index_loss × threshold` but `< index_loss` | Medium — consider relaxing trigger |
| **Far miss** | `bond_gain < index_loss × threshold` | **High — thesis violated; bonds failed as hedge** |

- Threshold is configurable (default: `0.90` — bonds covered at least 90% of index loss)
- Near-miss events surfaced to help Daniel decide if the redeployment trigger is too strict
- Far-miss events flagged with market context (VIX level, rate environment) — these are regime failures where stocks and bonds fell simultaneously (e.g., 2022 rate hike cycle)
- Report notes for each event: did the system issue a bond-sell recommendation? What was the gap between bond gain and index loss?
- **"Both stopped" scenario** (VUG and TLT stopped simultaneously or in close proximity): redeployment prompt is suppressed — no bond gains to compare. System shifts to VUG-only re-entry context nightly. When VUG re-entry signal fires, system presents paired recommendation: re-enter VUG + re-enter TLT on the same day. TLT re-enters as hedge alongside VUG, not independently.
- **Post-MVP options noted**: (a) TLT-first re-entry if TNX momentum turns negative before VUG signal; (b) hold BIL during the gap period; (c) asymmetric VUG re-entry at reduced allocation initially.

**Strategy comparison mode**: run the same simulation against up to 3 strategies side-by-side across the same periods. Uses the same engine and same metrics as above — no separate system. Max 3 strategies enforced — error if more passed. Report presentation details (layout, column format, highlighting) deferred to PRD.

```
tradingbot simulate multi-period --strategies rsi_ma_v1 rsi_ma_v2 momentum_v1
```

**Post-MVP — additional simulation reports:**
- **3/6/12 month live review**: periodic report on actual trading performance (not simulation) — number of trades, avg hold, avg gain over the period. Enables comparison of live performance vs simulation expectations.
- **Holding duration analysis**: would holding positions longer have produced higher P&L? Requires defining "longer" relative to actual exit — complexity deferred.
- **Trading frequency deviation**: strategy defines expected trade frequency; report tracks actual vs expected, flags sustained under/over-trading.
- **Gains deployment policy**: when a stock trade closes profitably, should proceeds move to ETF (VUG) or sit in cash? Daniel leans toward ETF or cash equivalent. Policy to be defined before implementation.

### 5.12 Perfect Trade Benchmark

- **Definition**: buy at the lowest price in the period, sell at the subsequent highest price
- **Capture ratio**: `realized_return ÷ perfect_return × 100%`
- Reported alongside actual and system P&L to give aspirational upper bound

### 5.13 Rebalancing Calculator

- Calculates drift from target allocations and suggests rebalancing trades
- Prices fetched automatically via Data Fetcher — no manual input
- Rebalancing events modeled as part of multi-period simulation — same component, historical data fed one day at a time
- Operational report reuses the same logic: `python portfolio.py rebalance --preview`
- Run on quarterly cadence (non-blocking nightly notification) + manual command

---

## 6. Portfolio Structure

The system generates recommendations and stop alerts. Daniel executes all orders manually in Fidelity. The system records outcomes via journal entries created after execution.

| Bucket | Allocation | Stop type | Stop level |
|--------|-----------|-----------|-----------|
| Index (VUG or similar) | 70% | Trailing | YTD gain protection formula |
| Bonds (TLT) | 25% | Trailing | 15% from purchase high |
| Active trades | 5% | Fixed | Per-trade (ATR-based, 2% risk rule) |

**ETF entry strategy (MVP):** Buy to target allocation on first run — no entry signal required. The portfolio composition decision (70/25/5) is the strategy for ETFs. Once in position, rebalancing and stops manage the position from there. Entry signal logic for ETFs (e.g., wait for price above 200MA) is post-MVP and should be validated through simulation before adopting.

**Index trailing stop formula:** if `ytd_return > 0`: protect 50% of gains (`entry × (1 + ytd_return × 0.50)`); else: 3% max loss floor (`entry × 0.97`). See `docs/data-model-sketch.md`.

**Bond stop**: 15% trailing stop from purchase high. Rationale: TLT's normal rate-driven fluctuations are ±5-10%; 15% catches structural regime failures (e.g., 2022 rate hike cycle) without triggering on noise.

**Active trade capital-at-risk cap**: 5% of total portfolio value across all active positions combined.

**Bond redeployment trigger rule**: When the index stop triggers, prompt redeployment of bond gains only when:
```
bond_gain_since_stop_date >= index_loss_at_stop  (both in $ terms)
```
See `README.md` for full bonds rationale and crash insurance logic.

---

## 7. Technical Constraints

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | Python | Daniel's primary language |
| Storage | SQLite + Alembic | Simple, offline, versioned schema |
| Historical prices | Gzipped CSV (`.csv.gz`) | Compact, append-only, git-excluded |
| Market data | yfinance `auto_adjust=True` | Handles splits/dividends automatically |
| Calendar | `pandas_market_calendars` | Accurate trading day validation |
| UI | Terminal prompts | Most natural for a developer-trader |
| `max_history_years` | Config value, default 5 | Controls total fetch window and simulation depth; applies to re-fetch cap and new ticker onboarding |
| Concurrency | None | Read-all → compute → write-once pattern |
| Cloud | None | Fully offline |
| Live broker | Fidelity (+ TradingView paper trading) | Daniel's existing accounts |
| LLM exchange format | JSON (SQLite); TOON noted for future | TOON (Token-Oriented Object Notation) deferred |
| Strategy definition | `docs/research/` folder | Same project, no separate repo |

**Strategy folder structure:**
```
docs/
  research/
    strategy-ideas.md
    indicators/
    observations/
```

---

## 8. CLI Conventions

**Unified CLI** (`tradingbot <subcommand>`) — consistent entry point, thin wrapper over importable modules. Each subcommand maps to a module concern.

| Command | Purpose |
|---------|---------|
| `tradingbot strategy dry-run --ticker AAPL --date 2026-03-09` | Single-day dry run |
| `tradingbot journal report --period=90d` | Generate all reports at once |
| `tradingbot simulate multi-period --portfolio sim_portfolio.yaml --years 5 --window 6m` | Multi-period simulation (automated) |
| `tradingbot simulate replay --portfolio sim_portfolio.yaml --start 2024-01-01` | Interactive replay |
| `tradingbot simulate replay --portfolio sim_portfolio.yaml --start 2024-01-01 --auto` | Automated single-run replay |
| `tradingbot journal compare --period 2025 --strategies rsi_ma_v1 rsi_ma_v2 momentum_v1` | Strategy comparison (max 3) |
| `tradingbot portfolio rebalance --preview` | Rebalancing calculator |
| `tradingbot nightly` | Run nightly review |

Report filenames include the date range: `override-impact-2026-01-08-to-2026-03-08.csv`

**Module structure:**
```
tradingbot/
  data/           # DataFetcher, MarketSnapshot construction
  strategy/       # Strategy, Metric, Indicator, Signal, Rule, RuleContext
  journal/        # JournalWriter, JournalReader, Alembic migrations
  portfolio/      # PortfolioManager, PositionScorecard, RebalancingCalculator
  simulation/     # ReplayEngine (shared by interactive, automated, multi-period)
  reports/        # All report generators, shared output formatting function
  config/         # portfolio.yaml loader, config hash enforcement
  tests/
    golden/       # Static golden input files (OHLCV, indicator values, portfolio states)
  tradingbot.py   # Unified CLI entry point
```

---

## 9. Hard Failures (No Fallbacks)

- ATR of zero — hard failure
- VIX or SPY fetch fails — hard failure
- Rule condition throws exception — hard failure (not silently suppressed)
- Fewer than 220 trading days available after fetch attempt — hard failure
- Price continuity check fails after corporate event re-fetch — hard failure (unadjusted split suspected; manual intervention required)
- **Cold-start behavior** (`JournalReader` with no history): methods like `win_rate`, `consecutive_losses`, `override_rate` return neutral defaults (e.g. `0.0`, `0`, `0.0`) when fewer than `n` records exist — never fail. Neutral defaults are surfaced transparently in output (e.g. "win rate: n/a — insufficient history").

---

## 10. Exit and Re-Entry Rules

### Active Stocks

| Event | Rule |
|-------|------|
| Stop hit | Exit at stop price (or gap fill price) |
| Target hit | Exit at target price |
| **Re-entry after stop** | **5 trading day cooldown — no re-entry on same ticker regardless of signal** |
| Re-entry after cooldown | Normal signal evaluation resumes |

- Cooldown length is configurable in strategy YAML (`reentry_cooldown_days: 5`)
- Cooldown is per-ticker, not portfolio-wide
- Post-MVP: evaluate replacing or layering cooldown with an `is_high_confidence` signal gate — but `is_high_confidence` flag needs more thought before it can serve this role reliably

### Index ETF (VUG)

| Event | Rule |
|-------|------|
| Stop hit | Exit (YTD gain protection formula triggered) |
| **Re-entry** | **Manual decision — no automatic re-entry signal** |

Every night after a VUG stop, the nightly review surfaces re-entry context until Daniel re-enters:

```
# VUG — STOPPED OUT (Day 8)

Re-entry signals:
  Price recovery:  VUG $241.20 | 200MA $248.50 | Below (-3.0%)
  Bond crossover:  Bond gain $1,840 | Index loss $2,100 | Gap -$260 (near miss)
  Momentum:        5d: -1.2% | 20d: -4.8% | 50d: -8.1%
                   → All negative — downtrend likely continuing

Decision: re-enter / wait
Reason (text): ___
```

- **Price recovery**: VUG price vs 200MA and vs stop price that triggered exit
- **Bond crossover**: bond gain vs index loss in $ terms (triggered / near miss / far miss). Suppressed if TLT is also stopped out — "both stopped" state displayed instead
- **Momentum**: 5d, 20d, 50d returns. 5d/20d reuse existing `Recommendation` fields; 50d is an additional calculation on existing OHLCV
- Plain-language momentum interpretation surfaced to reduce cognitive load
- Decision + reason recorded every night — creates accountability trail for re-entry timing
- P&L benchmarking not applied to index re-entry timing (less relevant for long-term holds)

### Bonds ETF (TLT)

| Event | Rule |
|-------|------|
| Stop hit | Exit (15% trailing stop from purchase high triggered) |
| **Re-entry** | **Manual decision — no automatic re-entry signal** |

Every night after a TLT stop, the nightly review surfaces re-entry context until Daniel re-enters:

```
# TLT — STOPPED OUT (Day 12)

Re-entry signals:
  Price recovery:  TLT $88.20 | 15% stop level $91.40 | Still below (-3.5%)
  Momentum:        5d: +0.4% | 20d: -3.1% | 50d: -9.2%
                   → Mixed — short-term stabilizing, medium-term still negative
  Rate direction:  10Y yield 20d momentum: +0.8% → rates still rising

Decision: re-enter / wait
Reason (text): ___
```

- **^TNX** (10-year Treasury yield) fetched via yfinance — no new data source; added to Data Fetcher scope
- `TNX_20d_momentum > 0` = rates still rising → TLT re-entry premature
- `TNX_20d_momentum < 0` = rates falling/stabilizing → TLT recovery more likely
- Decision + reason recorded nightly for accountability trail

**Post-MVP — Momentum direction change detection** (applies to both TLT and VUG re-entry):
- Surface when momentum is still negative but improving — "second derivative" signal indicating a potential turning point
- Implementation: `momentum_velocity = momentum_today - momentum_N_days_ago`
- If `momentum < 0` but `momentum_velocity > 0` → flag as "decelerating downtrend — watch for reversal"
- Useful as an early warning before the momentum signal itself turns positive

**Post-MVP — VUG substitute during high-volatility periods**:
- During sustained high VIX environments, substitute VUG with a lower-beta index instrument
- Candidate substitutes: SPLV or USMV (low-volatility factor ETFs — same equity exposure, lower drawdowns)
- Switching signal candidate: VIX level or VIX 20d trend (already fetched)
- Implement only after simulation confirms the substitution improves risk-adjusted returns

**Post-MVP — TLT/BIL rate-regime switching** (implement only after simulation confirms TLT far-miss frequency warrants it):

Priority order for switching signal:
1. Manual `fed_stance` entry after each FOMC meeting (`hiking | pausing | cutting`) — 8x/year, zero new infrastructure, Daniel already has a view
2. ^TNX 20d momentum as automated proxy — same data source, no new dependency
3. CME FedWatch implied probabilities — last resort; no official API, already priced into TLT price action, not worth the data source complexity

When `fed_stance = hiking` or `TNX_20d_momentum > 0`: hold BIL instead of TLT

---

## 11. Open Questions / Deferred

- ~~Q6~~ **RESOLVED**: Ambiguous exit (same-day stop + target) — fetch intraday data to determine sequence; hard failure if fetch fails.
- ~~**Exit and re-entry strategies**~~ **RESOLVED**: Defined in section 10 for active stocks, index ETF (VUG), and bonds ETF (TLT).
- TOON format: noted as future LLM exchange format; JSON used for now

---

## 12. Elicitation Methods Completed

| Method | Key Findings |
|--------|-------------|
| First Principles | Root cause is missing feedback loop, not bad strategy |
| Pre-mortem (Round 1) | Nightly friction, override rationalization, data gaps |
| 5 Whys | Reframed problem: accountability gap, not strategy gap |
| What If Scenarios | Earned autonomy model; stopping rules |
| SCAMPER | Terminal prompts over file editing; persistent recommendation flag |
| Occam's Razor | Simplified override tracking to single record + derived delta |
| Stakeholder Round Table | Portfolio value definition; capital-at-risk cap; scorecard for trim |
| Pre-mortem (Round 2) | Execution confirmation gap; fill simulation rules; gap-down formula |
| Self-Consistency Validation | Two-layer strategy design; N-ticker design from day one |
| Improv Yes-And | Momentum in recommendation; position scorecard stats; high-confidence flag |
| Inversion | Audit trail; strategy comparison report |
| Analogical Reasoning | Perfect trade benchmark; capture ratio |
| Future Backward | Go-live criteria; multi-period simulation |
| Extreme Cases | Bond allocation; index stop formula; rebalancing calculator |
| Critical Perspective | Terminal prompts confirmed; 5% active cap confirmed; index handled by system |
| Socratic Questioning (#41) | Multi-period simulation cadence; quarterly rebalancing; bonds rationale; Q6 resolved (intraday fetch) |
| Rubber Duck Debugging Evolved (#21) | Settlement prompt flow; unified trade lifecycle; 3-tag override system (N/R/G) with required comment; ranking formula with EV + Conviction alternatives |
| Algorithm Olympics (#22) | Ranking formula: R/R × win_rate for MVP; EV and Conviction computed alongside for periodic report comparison |
| Random Input Stimulus (#28) | Confidence signal red/yellow/green (post-MVP); LLM-assisted journal analysis (post-MVP); pre-nightly checklist (post-MVP) |
| Genre Mashup (#30) | LLM clinical summary + guided retrospective (post-MVP); pre-nightly checklist confirmed |
| Comparative Analysis Matrix (#33) | Python class + YAML config validated — all strategy parameters expressible in YAML |
| Critique and Refine (#42) | Settlement phase rewritten; open questions updated; win probability source linked to JournalReader; elicitation log updated; CSV/prompt terminology unified |
| Explain Reasoning (#43) | Recovery flow for missed nights (step-through each day); min_history_days configurable per strategy; accountability loop assumption made explicit |
| Lessons Learned (#50) | Real product = behavioral accountability tool; nightly flow is highest-risk implementation area; simplicity chosen repeatedly; post-MVP backlog is a roadmap; data model is the stable foundation |
