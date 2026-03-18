# Samaela Runbook

## Install

```bash
python -m pip install -e .[dev]
```

## Generate Deterministic Artifacts

```bash
python -m samaela.cli.fetch_data --config configs/backtest.template.yaml --output fixtures/universe_1h_binance.csv
python -m samaela.cli.run --config configs/backtest.template.yaml --mode backtest --seed 42
python -m samaela.cli.run --config configs/backtest.template.yaml --mode dry-run --seed 42
```

`python -m samaela.cli.fetch_data` now does two things in one pass:
- refreshes `artifacts/binance_universe.json` if the Binance universe cache is missing or older than 4 hours
- fetches candles for `long_symbol` plus the current top-10 eligible Binance USDT perpetual candidates ranked by `quoteVolume`

If you want the bot to rescan Binance every 4 hours, rerun the fetch command on that cadence from your scheduler of choice.

The current sample backtest artifact is schema-valid and currently has `validation.passed=true`.
The current paper-mode dry-run artifact is also schema-valid and currently shows a real six-persona hosted review veto blocking the trade.

## Validate

```bash
python -m samaela.cli.validate --file artifacts/runs/backtest_42.json --schema schemas/backtest_result.schema.json
```

## Dashboard

```bash
streamlit run apps/dashboard.py
```

## Safety Notes

- Keep `.env` local only.
- Keep live mode disabled unless 2FA confirmation and operator review are complete; this repo still blocks real live placement.
- Keep in mind that the widened short universe is now driven by the cached Binance snapshot rather than a permanently pinned static pair list.
- Hosted paper-mode review is now live: if hosted consensus is missing, the trade blocks; if hosted consensus vetoes, the trade also blocks.
- Review the PR template and backtest diff before approving any change.
