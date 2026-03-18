# Samaela Risk Policy

Samaela operates in deterministic backtest and testnet-first dry-run modes by default. Live order placement and websocket streaming remain blocked stubs until explicit exchange wiring is added.

## Account Controls

- Margin mode defaults to isolated.
- Live execution requires config opt-in, `SAMAELA_ENABLE_LIVE=1`, and `SAMAELA_OPERATOR_2FA_OK=1`, but the current repository still blocks actual live placement.
- Position sizing uses a half-Kelly cap, confidence scaling, and liquidation buffer checks.

## Execution Controls

- Binance request weight is rate-limited with a 1200 weight-per-minute token bucket.
- TWAP slicing uses the configured `twap_hours` value.
- Hosted LLM review is optional for deterministic backtests, but paper/testnet mode can require a real hosted persona panel.
- When hosted paper-mode review is expected and consensus is missing, the trade is blocked rather than allowed through a local fallback.
- When the hosted persona panel issues a veto, the paper/testnet trade is blocked.

## Validation Controls

- Walk-forward backtests are mandatory.
- Monte Carlo, CPCV, and PBO are recorded in every backtest artifact.
- Reproducibility hashes and chain-audit stubs are written for every run.
