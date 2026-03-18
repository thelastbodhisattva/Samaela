# Audit Stub Contracts

Samaela writes a local JSONL audit log and a deterministic `chain_stub_tx` hash for each run.

## Required Fields

- `run_id`
- `output_path`
- `audit_log_path`
- `chain_stub_tx`

## Intended Follow-On Integrations

- On-chain notarisation sink
- External archive storage
- PR comment bot that posts backtest diffs and hashes
