# F08 · S04 — Multi-Stage Cross-Cell Production Chains

**Feature:** F08 — Macro Manufacturing
**Scope:** macro · **MVP:** yes · **Depends on:** S03, F07 S04

## Intent
Compose mines, roads, and manufacturers into multi-stage chains across the map — the
payoff of the macro layer: the map itself becomes a factory graph producing
higher-`Tier` resources.

## Behavior
1. Resources flow `mine → road → manufacturer → road → manufacturer → …`, each stage
   raising tier per its recipe. Any number of stages may be chained across cells.
2. The chain reaches a deterministic steady-state throughput limited by its slowest
   stage (the bottleneck — slowest miner, road `rate`, or manufacturer `craftTime`).
3. Back-pressure propagates upstream through the whole chain; starvation propagates
   downstream — both deterministically and without material loss.

## Data touched
Multiple cells' `MacroManufacturer`, `RoadSegment`, `MacroOutput` across the chain.

## Acceptance criteria
- **AC-1** *Given* a two-stage chain, *then* steady-state output equals the bottleneck
  stage's rate within ε, identical across runs (F08-A3).
- **AC-2** *Given* a downstream stall, *then* back-pressure reaches the source and halts
  it with no loss; clearing the stall resumes the chain.
- **AC-3** *Then* total material is conserved across the entire chain over a long run.

## Out of scope
Per-stage recipe math (S02); blueprint replication of chains (F09 S04).
