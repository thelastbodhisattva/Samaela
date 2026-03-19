# Bidirectional Spread Engine

## Overview

The Bidirectional Spread Engine is Samaela's current spread-selection and routing model for anchor-versus-candidate trades.

It keeps `BTCUSDT` as the anchor symbol in the current main template, but it no longer assumes the anchor must always be the long side of the trade. Instead, the engine can express either side of the spread depending on where the current deviation sits relative to the estimated equilibrium.

In practice, that means the system can now choose between:

- `long_anchor_short_candidate`
- `short_anchor_long_candidate`

## Why We Changed It

The earlier engine was effectively one-way: it treated the anchor as the fixed long leg and the candidate basket as the fixed short side.

That had two major weaknesses:

1. It left valid mean-reversion opportunities unused whenever the spread signal implied the opposite side.
2. It forced the strategy to behave more like a directional posture than a true spread engine, even when the statistics clearly supported the inverse expression.

The bidirectional model fixes that by letting the spread sign determine the trade orientation while still preserving the same statistical gates, hosted review contract, and deterministic backtest pipeline.

## What It Does

- Evaluates the anchor and each candidate over the configured lookback window.
- Estimates an equilibrium spread using a linear relationship between candidate and anchor.
- Measures current deviation with a z-score.
- Converts the sign of that z-score into a spread direction.
- Keeps only one coherent direction in the final shortlist so the basket does not mix opposing spread bets.
- Routes the anchor and candidate sides through hedge construction, margin planning, execution simulation, and trade recording with that direction preserved.

## How Direction Is Chosen

The signal engine computes a spread and a z-score for every candidate.

- Positive z-score -> `long_anchor_short_candidate`
- Negative z-score -> `short_anchor_long_candidate`

The current implementation lives in `src/samaela/core/signal_pairs.py`.

After candidate scoring, the local decision builder takes the top valid direction and keeps only candidates aligned with that same side of the spread. This prevents the engine from building a mixed basket where one leg expects anchor strength and another expects anchor weakness at the same time.

## Selection Pipeline

The Bidirectional Spread Engine still uses the same statistical discipline as the rest of the system.

For a candidate to qualify, it must pass the configured filters for:

- cointegration p-value
- ADF p-value on the spread
- correlation stability
- finite half-life under the configured maximum
- nonzero spread richness

So the direction is flexible, but the statistical gate is not looser.

## Hedge Construction

Once a direction is chosen, the hedge optimizer maps that direction into actual order sides.

- If the spread direction is `long_anchor_short_candidate`, candidate hedge legs are `SELL` and the anchor is `BUY`.
- If the spread direction is `short_anchor_long_candidate`, candidate hedge legs are `BUY` and the anchor is `SELL`.

This behavior is implemented in `src/samaela/core/hedge_optimizer.py`.

The optimizer still solves for weights with the same risk bounds and exposure controls. What changed is the side assignment, not the idea of weighted hedge construction itself.

## Execution Behavior

The execution router preserves the selected sides on entry and reverses them on exit.

That means the bidirectional decision is not just metadata inside the AI payload. It is carried all the way into the simulated order list and the realized trade record.

The current execution path also keeps the volatility-aware and concentration-aware slippage model that was added in the same broader improvement cycle.

Execution behavior lives in `src/samaela/core/execution_router.py`.

## What Changed In The Data Contracts

The main AI decision payload now includes `pair_directions` alongside `ranked_pairs` and `weights`.

That contract is enforced in `schemas/ai_decision.schema.json`, where each direction must be one of:

- `long_anchor_short_candidate`
- `short_anchor_long_candidate`

Trade records and hedge baskets also carry side-aware fields so artifacts reflect the actual spread expression rather than implying the old one-way assumption.

## What Stayed The Same

- The hosted six-persona vote panel remains the final decision interface.
- The main signal gate still depends on cointegration, stationarity, correlation stability, half-life, and confidence.
- The engine still normalizes to one coherent shortlist before hedge construction.
- Risk, slippage, validation, and persona evolution gates remain separate controls.

So this was not a rewrite into a fully different strategy family. It was a correction to how the spread was expressed.

## Earlier Version vs Current Version

| Area | Earlier fixed-direction engine | Current bidirectional engine |
| --- | --- | --- |
| Anchor assumption | Anchor effectively stayed long | Anchor can be long or short |
| Candidate side | Candidate basket effectively stayed short | Candidate side flips with spread direction |
| Spread expression | One-way | Two-way |
| AI contract | Pair ranking and weights only | Pair ranking, weights, and explicit `pair_directions` |
| Hedge routing | Direction implied by old posture | Direction carried explicitly into basket sides |
| Exit handling | Reverse of one-way entry | Reverse of the actual selected entry direction |

The main practical difference is that the current engine can express both halves of a mean-reversion setup instead of only the anchor-long half.

## Why This Matters Operationally

- It makes the spread engine closer to the actual signal math.
- It avoids discarding inverse opportunities that still pass the statistical gate.
- It makes artifacts, schemas, and execution reports more honest about which side of the spread the system intended to trade.
- It reduces the mismatch between local deterministic signal generation and hosted review context.

## Current Caveats

- Bidirectionality does not guarantee higher trade count in every mode. Execution mode still evaluates only the latest bar, so a dry-run can legitimately return no trade even when the backtest found earlier windows.
- Live mode remains blocked elsewhere in the repo, so bidirectional routing currently matters for backtests and dry-run simulation, not real exchange placement.
- The main winning template is still intentionally narrow and only uses a pinned shortlist in production-style backtests. Bidirectionality improves expression of the signal, but the repo still keeps strict confidence and validation gates.

## Related Files

- `src/samaela/core/signal_pairs.py`
- `src/samaela/core/hedge_optimizer.py`
- `src/samaela/core/execution_router.py`
- `src/samaela/core/types.py`
- `schemas/ai_decision.schema.json`
- `README.md`
