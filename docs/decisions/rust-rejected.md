# Decision: Python over Rust for Simulation Engine

**Date:** 2026-03-12
**Status:** Decided — Python retained

---

## The Question

During product brief review, Daniel raised whether the simulation engine should be implemented in Rust (or use concurrency) to handle multi-period backtesting performance.

## Why It Was Raised

Multi-period simulation runs 5 years × 6-month rolling windows across a portfolio of instruments. Concern: Python might be too slow for this workload.

## Why Rust Was Rejected

**The workload doesn't warrant it.** The actual computation is:
- 10 simulation windows (5 years × 6-month periods)
- ~126 trading days per window
- ~7 instruments per day
- ~900 strategy evaluations total per full run
- Each evaluation = pandas rolling calculations (MA, RSI, ATR) — highly optimized, vectorized

Python handles this in seconds to low minutes. There is no performance problem to solve.

**The ecosystem cost is too high.** Python has yfinance, pandas, numpy, Alembic, and a mature data science stack. Rust has none of these. Switching means rebuilding data infrastructure from scratch, not just rewriting the simulation loop.

**Debugging cost without expertise is real.** LLMs write competent Rust, but borrow checker errors are non-trivial to diagnose without Rust experience. For a personal project that will evolve over time, this raises maintenance cost considerably.

**Daniel's primary language is Python.** Long-term ownership and iteration matter more than peak throughput for a single-user offline tool.

## What to Do Instead

If simulation performance ever becomes a real bottleneck (measurable, not hypothetical):
1. Profile first — identify the actual slow path
2. Add `multiprocessing` for parallel window execution — one line of change, covers the realistic concern
3. Vectorize any remaining loops in pandas/numpy

## When to Revisit

Only if profiling shows simulation taking more than a few minutes on realistic data, AND the bottleneck is CPU-bound computation (not I/O or data loading). That scenario is unlikely given the scale of this project.
