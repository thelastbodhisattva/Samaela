# Persona Evolution Engine

## Overview

The Persona Evolution Engine is the local adaptive layer that keeps Samaela's six analyst personas current as market conditions change.

It does not replace the six hosted analysts or change their identities. Instead, it updates the local context prefix injected ahead of the hosted prompt so the same panel can respond with regime-aware and evidence-aware context.

The six personas remain:

1. Conservative Analyst
2. Volatility Hunter
3. Macro Trader
4. Quant Optimizer
5. Contrarian Skeptic
6. Smart Money Whale

## What It Does

- Keeps the six personas fixed in identity and order while allowing their local context to evolve.
- Stores persona state as snapshot artifacts with a reproducibility hash, Git commit hash, timestamp, regime, and lifecycle metadata.
- Uses validated backtest evidence to update per-regime persona beliefs and local meta-weights.
- Injects the active local prefix block into the hosted decision prompt before the six-persona vote is generated.
- Preserves a safe rollback point so a degraded evolution can be reversed automatically or manually.

## What Stays The Same

- The hosted model still generates the final six-persona vote panel.
- The vote schema, persona identities, persona order, and consensus contract stay fixed.
- The personas still operate as a democratic analyst panel rather than as six independent trading bots.
- The validation gate remains the authority for whether a new evolved snapshot can be promoted.

## What Changed From The Earlier Static Personas

| Area | Earlier static version | Current evolved version |
| --- | --- | --- |
| Persona identity | Fixed six personas | Same fixed six personas |
| Prompt context source | Static prompt text | Active snapshot-backed local prefix |
| Update timing | No local evolution | Only after validated backtest evidence passes the gate |
| Regime awareness | Implicit only | Explicit active regime and regime-specific prefix |
| Failure behavior | No snapshot lifecycle | Failed or incomplete validation prevents promotion |
| Rollback | None | Safe snapshot pointer and automatic/manual rollback |
| Audit trail | Limited to run outputs | Snapshot metadata with commit hash, reproducibility hash, timestamps, and lifecycle |

The most important difference is simple: earlier personas were fixed prompts, while the current personas keep the same identities but receive a locally evolved prefix based on validated historical evidence.

## How It Works

### 1. A validated backtest artifact is produced

The engine only considers artifacts that already passed the required robustness checks.

That means the backtest must contain nonzero walk-forward evidence, nonzero Monte Carlo evidence, nonzero CPCV evidence, and a `PBO` below the configured threshold.

### 2. The local engine infers the active regime

The snapshot stores an active regime label such as `bull`, `bear`, `sideways`, or `high-vol`.

That regime becomes part of the persona prefix so each analyst can be reminded how its historical edge compares with baseline in the current regime.

### 3. Persona beliefs are updated

Each persona keeps per-regime belief state. Current behavior is driven by Bayesian-style belief updates plus deterministic meta-weight adjustment.

This is the user-visible adaptive layer today. It is accurate to describe the system as evolving personas through validated evidence, but not accurate to describe it as fully deployed RL policy training for persona control.

### 4. A new snapshot is written

Each snapshot records:

- `version_id`
- `created_at`
- `evolved_at`
- `git_commit_hash`
- `reproducibility_hash`
- active regime
- validation gate summary
- performance summary
- lifecycle state
- per-persona beliefs, active prefix, posterior probability, and meta-state

### 5. The active safe pointer may move

If the new snapshot is acceptable, it becomes the active safe snapshot.

If it degrades too much versus the current safe snapshot, the engine keeps or restores the safe one instead.

### 6. The hosted prompt receives the active prefix block

When Samaela asks the hosted model for the final vote, it prepends the active local persona block to the hosted decision prompt.

That means the hosted vote stays the final authority, but it now reads locally evolved context before answering.

## Evolution Gate

Evolution is intentionally strict.

A snapshot is only eligible for promotion after explicit validated evidence is present:

- walk-forward windows > 0
- Monte Carlo paths > 0
- CPCV combinations > 0
- `PBO < 0.05`

If those conditions are not met, the artifact can still exist, but persona evolution does not promote a new snapshot.

## Snapshot Lifecycle

The local persona state keeps:

- an active snapshot pointer
- a safe snapshot pointer
- a snapshot directory with retained historical versions

Snapshots are pruned to a small retained set while preserving the active safe rollback checkpoint.

This is how the system avoids unbounded drift while keeping a known-good recovery point.

## Rollback And Lock Behavior

### Safe rollback

If the candidate snapshot degrades too far against the current safe snapshot, it is rolled back automatically instead of being promoted.

The current implementation compares performance characteristics such as Sharpe degradation and drawdown worsening before declaring the candidate safe.

### 7-day lock

Even a valid artifact cannot promote a new snapshot if the 7-day wall-clock evolution lock is still active.

This prevents the persona state from thrashing on every short-term run.

### Manual control

Operators can use `evolve_personas.py` for manual promotion or manual rollback:

```bash
python evolve_personas.py --config configs/backtest.template.yaml --artifact artifacts/runs/backtest_42.json
python evolve_personas.py --config configs/backtest.template.yaml --rollback <snapshot_id>
```

## Operational Caveats

- The hosted model is not retrained by this system; the hosted final vote remains the same interface and authority.
- Current user-facing behavior is local prefix evolution plus Bayesian/meta-weight adaptation. It should not be documented as fully trained live RL persona control.
- Failed or incomplete validation prevents promotion.
- The 7-day lock can block promotion even when the artifact itself is valid.
- Missing analyst-vote evidence can leave meta-weights unchanged while still allowing an auditable checkpoint.
- Live execution remains blocked elsewhere in the repo; persona evolution is not what prevents live trades today.

## Why This Is Better Than The Earlier Static Setup

The earlier static six-persona setup had one major limitation: the identities stayed fixed and so did the local context given to them.

The current approach keeps the benefits of a stable six-analyst panel while adding:

- validated regime awareness
- explicit snapshot history
- rollback safety
- deterministic auditability
- promotion locks
- better continuity between backtest evidence and future hosted decisions

In practice, that means Samaela can preserve the same democratic panel while adapting the context those analysts receive, instead of pretending that market structure never changes.

## Related Files

- `src/samaela/core/persona_state.py`
- `src/samaela/core/llm_contracts.py`
- `src/samaela/core/validation.py`
- `evolve_personas.py`
- `prompts/ai_decisioning_system.md`
