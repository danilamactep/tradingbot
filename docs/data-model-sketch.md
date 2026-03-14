# Data Model & Class Sketches

> **Status**: Pre-architecture sketches — captured during product brief elicitation.
> These are not authoritative. The architecture doc (to be created) supersedes this file.

---

## Strategy Engine — Core Abstractions

```python
@dataclass
class Metric:
    name: str
    def calculate(self, snapshot: MarketSnapshot) -> float: ...

@dataclass
class Indicator:
    name: str
    def evaluate(self, metrics: dict[str, float]) -> bool: ...  # threshold from YAML config

@dataclass
class Signal:
    name: str
    indicators: list[Indicator]
    def triggered(self, states: dict[str, bool]) -> bool: ...
    def action(self) -> str: ...  # 'buy' | 'sell' | 'hold' | 'no-action'

@dataclass
class Rule:
    name: str
    condition: Callable[[RuleContext], bool]
    message: str
    severity: str  # 'info' | 'warning' | 'block'

@dataclass
class RuleContext:
    snapshot: MarketSnapshot
    metrics: dict[str, float]
    journal: JournalReader  # read-only SQLite wrapper

class Strategy:
    name: str
    metrics: list[Metric]
    indicators: list[Indicator]
    signals: list[Signal]
    rules: list[Rule]

    def evaluate(self, snapshot: MarketSnapshot) -> Recommendation:
        metric_values = {m.name: m.calculate(snapshot) for m in self.metrics}
        indicator_states = {i.name: i.evaluate(metric_values) for i in self.indicators}
        signal = next((s for s in self.signals if s.triggered(indicator_states)), None)
        ctx = RuleContext(snapshot=snapshot, metrics=metric_values, journal=...)
        # apply rules, build Recommendation ...
```

---

## Data Model — Key Dataclasses

```python
@dataclass
class MarketContext:
    as_of_date: date
    mode: str                    # 'replay' | 'paper' | 'live'
    spy_daily_return: float
    vix_level: float
    risk_basis: float            # cash + cost basis of open positions — used for sizing/cap enforcement
    market_value: float          # risk_basis + unrealized_pnl — actual account value
    available_cash: float
    total_capital_at_risk: float
    capital_at_risk_pct: float
    all_positions: list[Position]

@dataclass
class MarketSnapshot:
    ticker: str
    ohlcv: pd.DataFrame          # 220 validated trading days
    reference_index: str         # e.g., "NLR", "SOXX", "QQQ"
    reference_ohlcv: pd.DataFrame
    position: Position | None
    market: MarketContext

@dataclass
class Recommendation:
    action: str                  # 'buy' | 'sell' | 'hold' | 'no-action'
    entry: float
    stop: float
    target: float
    risk_reward: float
    stop_distance_pct: float
    momentum_5d: float
    momentum_20d: float
    reference_index_return: float
    relative_strength: float     # ticker vs reference index
    signal_description: str
    rules_warned: list[str]
    rules_blocked: list[str]
    is_persistent: bool          # same recommendation 3+ consecutive days
    is_high_confidence: bool     # signal combo with above-average win rate
    reasoning: dict              # structured reasoning for audit trail
```

---

## Portfolio Config — `config/portfolio.yaml` Structure

```yaml
portfolio:
  starting_capital: 50000     # source of truth for cash baseline; overridable per simulation
  stock_allocation_cap: 0.02  # hard cap per ticker — policy applied to all stock instruments

  instruments:
    VUG:
      allocation: 0.70        # target allocation of total portfolio
      stop: ytd_protection    # presence of stop field → ETF, allocation-based sizing

    TLT:
      allocation: 0.25
      stop: trailing_15pct

    AAPL:
      reference_index: QQQ   # presence of reference_index → stock, ATR sizing, cooldown rules

    OKLO:
      reference_index: NLR

    NVDA:
      reference_index: SOXX
```

---

---

## Journal — Schema

```
portfolios:  id | name ("actual", "system", ...) | starting_capital | created_at
trades:      id | portfolio_id (FK) | ticker | date | entry | exit | stop | target | ...
reasoning:   id | trade_id (FK) | source ("system" | "human") | payload (JSON)
divergences: id | system_trade_id (FK) | actual_trade_id (FK) | type | reasoning_id (FK)
```

Adding a new portfolio type = one INSERT into `portfolios`. No schema migration, no code change.

**Reasoning payload — system:**
```json
{
  "snapshot_id": "AAPL_2026-03-09",
  "source_file": "historical_prices/AAPL.csv.gz",
  "source_hash": "sha256:a3f2c1d4...",
  "date_range": {"from": "2025-06-01", "to": "2026-03-09"},
  "rsi_14": 28.3, "ma_50": 181.20, "ma_200": 175.40,
  "atr_14": 8.20, "atr_50": 7.10, "vix": 22.1,
  "spy_1d_return": -0.012, "capital_at_risk_pct": 0.038,
  "rules_warned": ["high_vix"], "rules_blocked": []
}
```

**Reasoning payload — human:**
```json
{
  "tag": "G",
  "comment": "felt overextended given macro",
  "divergence_type": "ignored_sell"
}
```

## Journal — JournalReader Interface

```python
class JournalReader:
    def __init__(self, portfolio_id: int): ...   # works identically for any portfolio
    def win_rate(self, ticker: str, last_n: int) -> float: ...
    def consecutive_losses(self, ticker: str) -> int: ...
    def signal_win_rate(self, signal_name: str, last_n: int) -> float: ...
    def divergence_rate(self, last_n: int) -> float: ...
    def divergence_quality_score(self, last_n: int) -> float: ...  # % of divergences where Daniel beat system
    def divergence_pnl_by_tag(self, last_n: int) -> dict[str, float]: ...  # avg P&L delta per tag
```

---

## Position Scorecard

```python
@dataclass
class PositionScore:
    ticker: str
    unrealized_pnl_pct: float
    distance_to_target_pct: float
    days_held: int
    momentum_5d: float
    momentum_20d: float
    reference_index_return: float
    relative_strength: float
```

---

## Portfolio — Index Trailing Stop Formula

```python
if ytd_return > 0:
    stop = entry_price * (1 + ytd_return * 0.50)  # protect 50% of gains
else:
    stop = entry_price * 0.97  # 3% max loss floor
```
