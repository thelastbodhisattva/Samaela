# Public Documentation Package

This public repository currently contains documentation and example result artifacts only. The codebase, CLAUDE.md, and AGENT.md are intentionally excluded for now and may be published later.

# Samaela

Samaela is a deterministic, modular long-core and short-basket crypto trading bot for Binance USDT-M futures research, backtesting, and guarded dry-run execution. The core long comes from config, the hedge basket is selected from Binance-discovered candidates, all machine outputs are strict JSON validated against local schemas, and live execution remains a blocked stub behind explicit environment controls until real exchange wiring is implemented.

## What ships in this repo

- Exactly 10 core modules under `src/samaela/core/`
- YAML-driven configuration with `long_symbol`, leverage, margin mode, risk, and TWAP controls
- Deterministic backtests and dry-runs backed by Binance historical snapshots plus a cached 4-hour candidate-universe scan
- CVXPY basket optimisation, Statsmodels cointegration diagnostics, Monte Carlo, CPCV, and PBO validation
- OpenRouter structured-output integration with strict schema mode, prompt caching, and a process-level token bucket
- Streamlit operator dashboard, governance templates, reproducibility hashes, and audit-log stubs

## Repository layout

- `src/samaela/core/config.py` loads YAML plus `.env` and resolves runtime safety gates
- `src/samaela/core/market_data.py` loads fixtures, refreshes a cached Binance universe snapshot, and fetches USD-M historical klines directly
- `src/samaela/core/signal_pairs.py` ranks short candidates against the configured core long
- `src/samaela/core/hedge_optimizer.py` solves the hedge basket with CVXPY + OSQP/ECOS
- `src/samaela/core/risk_controls.py` applies half-Kelly sizing, margin checks, and liquidation buffers
- `src/samaela/core/execution_router.py` simulates TWAP execution with a Binance token bucket
- `src/samaela/core/backtest_runner.py` runs walk-forward backtests and execution previews
- `src/samaela/core/llm_contracts.py` enforces strict AI decision JSON and prompt-cache reuse
- `src/samaela/core/validation.py` handles canonical JSON, Monte Carlo, CPCV, PBO, and hashes
- `apps/dashboard.py` renders the operator dashboard from generated JSON artifacts

## Install

```bash
python -m pip install -e .[dev]
```

Copy `.env.example` to `.env` if you want to wire Binance or OpenRouter credentials locally.

## Historical data

The demo fixture `fixtures/universe_1h_binance.csv` is sourced from Binance USD-M futures klines rather than synthetic candles. The fetch workflow now refreshes `artifacts/binance_universe.json` lazily on a 4-hour TTL using Binance `exchangeInfo` plus `ticker/24hr`, ranks eligible USDT perpetuals by `quoteVolume`, excludes the configured core long, and keeps the top 10 candidates before pulling candles.

To refresh both the cached universe snapshot and the fixture directly from Binance, run:

```bash
python -m samaela.cli.fetch_data --config configs/backtest.template.yaml --output fixtures/universe_1h_binance.csv
```

## Configuration example

`configs/base.yaml` includes the requested operator fields:

```yaml
long_symbol: BTCUSDT
leverage: 5
risk_per_trade_pct: "2.0"
twap_hours: 4
liquidation_buffer_pct: "30.0"
margin_mode: isolated
```

`short_candidates` still exists in config as a compatibility fallback for offline fixture flows, but the main Binance fetch path now prefers the cached dynamic universe.

## Run

Backtest:

```bash
python -m samaela.cli.run --config configs/backtest.template.yaml --mode backtest --seed 42
```

The tuned sample backtest uses `BTCUSDT` as the core long, a Binance-refreshed top-10 short universe cached in `artifacts/binance_universe.json`, a pinned Binance 1h fixture window from `2026-03-08T01:00:00Z` to `2026-03-18T00:00:00Z`, and `0.5` bps slippage to represent a liquid futures demo assumption.

Dry-run:

```bash
python -m samaela.cli.run --config configs/backtest.template.yaml --mode dry-run --seed 42
```

Validate a generated artifact:

```bash
python -m samaela.cli.validate --file artifacts/runs/backtest_42.json --schema schemas/backtest_result.schema.json
```

Dashboard:

```bash
streamlit run apps/dashboard.py
```

## Safety model

- Testnet-first execution remains the default.
- Live mode needs config opt-in, `SAMAELA_ENABLE_LIVE=1`, and `SAMAELA_OPERATOR_2FA_OK=1`.
- Hosted LLM calls activate for low-confidence or explicitly allowed top-k review paths; paper-mode now supports a real six-persona hosted review panel.
- When hosted paper-mode review is expected, missing hosted consensus blocks the trade instead of silently falling back.
- A real hosted persona panel can veto a paper-mode trade, and the veto is enforced operationally.
- Live order placement and websocket streaming remain guarded stubs in this repo.
- Every run writes canonical JSON, a reproducibility hash, a local audit log entry, and a chain-audit stub hash.
- The shipped demo fixture is built from real Binance historical data and the cached universe scan is refreshed from live Binance metadata, not synthetic candles.
- Because the short universe is now refreshed from live Binance ranking data, the exact backtest outcome depends on the cached snapshot present in `artifacts/binance_universe.json` when you build or refresh the fixture.
- The current sample backtest artifact in `artifacts/runs/backtest_42.json` passes validation with `validation.passed=true`.
- The current paper-mode dry-run artifact in `artifacts/runs/dry-run_42.json` contains a real six-persona hosted review and is currently blocked because the hosted panel vetoed the trade.

## Governance

- PRs use `.github/pull_request_template.md`.
- Backtest diffs are documented with `governance/BACKTEST_DIFF_TEMPLATE.md`.
- Risk, audit, and decision-log expectations live in `governance/`.

Samaela is ready for deterministic backtest and dry-run operation.

One-line run command:

```bash
python -m samaela.cli.run --config configs/backtest.template.yaml --mode backtest --seed 42
```
