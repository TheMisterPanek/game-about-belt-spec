# F01 · S03 — Fixed-Point Clock & Rounding Contract

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
Make non-integer simulation quantities reproducible across languages by fixing the
numeric representation and the rounding rule. This is what lets a Rust and a Python
implementation agree exactly.

## Behavior
1. Simulation-affecting non-integer quantities (rates, progress, craft time
   accumulators) are `Fixed`: integer numerators over the global denominator
   **1024**. No other scale is used for simulation state (Invariant I6).
2. `rate × Δt` and similar products are computed in `Fixed` arithmetic with the
   documented operation order, so intermediate values match bit-for-bit.
3. Converting accumulated `Fixed` amount into discrete `Int` units uses **floor**;
   the fractional remainder stays in the source accumulator and is carried to the next
   tick. No unit is created or destroyed by rounding.
4. Real numbers may appear only in presentation (camera, interpolation) and never in
   any value read by a system.

## Data touched
All `Fixed` fields (`MinerState.rate`, `AssemblerState.progress`, `RateEntry.rate`,
`RoadSegment.rate`, …); discrete buffers they feed.

## Acceptance criteria
- **AC-1** *Given* a source producing at a fractional `Fixed` rate, *when* run for K
  ticks, *then* total discrete units emitted equals `floor(Σ rate×Δt)` with the
  remainder retained — verified by exact unit count.
- **AC-2** *Given* the same `Fixed` inputs, *then* two implementations compute the same
  numerator at the chosen scale (no platform float divergence).
- **AC-3** *Over* a long run, total units in = units out + units buffered + carried
  remainders; nothing is lost or invented.

## Out of scope
The choice of `Δt` value (S01); per-machine rates (F06/F02).
