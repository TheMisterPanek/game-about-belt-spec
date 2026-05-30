# F07 · S03 — Throughput Limits & Back-Pressure

**Feature:** F07 — Macro Routing: Roads & Big Belts
**Scope:** macro · **MVP:** yes · **Depends on:** S02

## Intent
Cap how fast a road can carry and make congestion propagate correctly, so a slow
downstream throttles upstream production instead of losing material.

## Behavior
1. Each `RoadSegment` has a max `rate` (`Fixed`, units/second). Per tick it moves at
   most `floor(rate × Δt)` units (carrying remainder, F01 S03).
2. If the downstream intake cannot accept (full buffer / blocked manufacturer), the
   road's `buffer` fills; once full it stops pulling from upstream, which in turn fills
   the source `MacroOutput.buffer` to capacity, stalling production (Invariant I2).
3. Back-pressure is deterministic and lossless: no units are dropped at any stage.

## Data touched
`RoadSegment.rate/buffer`, upstream `MacroOutput.buffer`, downstream intake.

## Acceptance criteria
- **AC-1** *Given* unlimited upstream supply, *then* delivered throughput equals the
  road's `rate` within ε (the road is the bottleneck).
- **AC-2** *Given* a blocked downstream, *then* the road fills, upstream fills, and
  production stalls — with zero unit loss (conservation).
- **AC-3** *When* the downstream unblocks, *then* flow resumes deterministically.

## Out of scope
Splitting at junctions (S04); manufacturer behavior (F08).
