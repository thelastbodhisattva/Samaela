# Changelog

## 2026-03-19

Documentation-only release.

This release does not introduce new runtime, schema, trading, or API behavior by itself. It documents the current shipped behavior already present in the repo and packages that explanation as markdown-only deliverables.

### Documented current features

- Added a dedicated `PERSONA_EVOLUTION_ENGINE.md` reference explaining how the six analyst personas now evolve through a local snapshot-based prefix system while the hosted six-persona vote contract stays the same.
- Documented the current bidirectional spread engine, where trades can route as either `long_anchor_short_candidate` or `short_anchor_long_candidate` instead of the earlier fixed one-way posture.
- Documented the current backtest `no_trade_diagnostics` contract, including the difference between `structural_no_window`, `no_signal`, `rejected_signal`, and `trades_executed` outcomes.
- Documented the shipped research configs used to reproduce no-trade cases and alias-backed fixture loading without changing the main winning backtest template.

### User-facing behavior already present in the repo

- The same six personas remain in the same order: Conservative Analyst, Volatility Hunter, Macro Trader, Quant Optimizer, Contrarian Skeptic, and Smart Money Whale.
- The hosted system still produces the final six-persona vote panel and consensus output.
- Persona evolution only promotes new local prefix snapshots after validated backtest evidence passes the gate.

### Operational caveats

- Persona evolution is blocked unless walk-forward, Monte Carlo, CPCV, and `PBO < 0.05` requirements are satisfied.
- A 7-day wall-clock lock can prevent a new snapshot from being promoted even when a fresh artifact is valid.
- Missing analyst-vote evidence can leave persona meta-weights unchanged even when the resulting checkpoint remains auditable.
- Live execution remains a guarded stub, and hosted veto behavior is unchanged.

### Scope of this release

- Markdown documentation only.
- No code, tests, schemas, artifacts, `CLAUDE.md`, `AGENT.md`, or other agent-control files are intended for the public docs-only commit.
