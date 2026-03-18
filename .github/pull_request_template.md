## Summary
- Explain the trading or validation change in one sentence.
- Link the config used for the comparison.
- Attach the backtest diff and reproducibility hash.

## Backtest Diff
- Base commit hash:
- Head commit hash:
- Output artifact path:
- Return delta:
- Max drawdown delta:
- PBO delta:

## Risk Checks
- Margin mode reviewed:
- Testnet path exercised:
- Live path remains guarded:
- 2FA operator confirmation procedure preserved:

## Verification
- `python -m pytest`
- `python -m samaela.cli.run --config configs/backtest.template.yaml --mode backtest --seed 42`
- `python -m samaela.cli.run --config configs/backtest.template.yaml --mode dry-run --seed 42`
- `python -m samaela.cli.validate --file artifacts/runs/backtest_42.json --schema schemas/backtest_result.schema.json`
