# Benchmark — Reference Scenarios

Concrete, deterministic scenarios run against the headless harness (`F01 S07`). Each
gives a setup, a command script (described abstractly), and the invariants/totals to
check. Parameters use the MVP catalog (`F02 feature.md`) and defaults
(`cellSize = 50`, `TICKS_PER_SECOND = 30`, `Fixed` scale 1/1024).

> Exact numeric targets are computed by the reference (all-Fine) run; the scenarios
> below specify the **structure** and the **properties** that must hold. A benchmark
> harness pins down the literal expected totals per target seed.

---

## S-GEN — Deterministic generation
**Setup:** `generate({seed: 42, 64×64, cellSize: 50, oreFinite: false})` twice.
**Check:** the two grids' dumps are identical (terrain, deposits, outputs) — `F03-A1`.
Changing the seed changes the grid.

## S-MINE — Single mining cell, fine vs coarse (core P-LOD)
**Setup:** one `Iron` cell developed with: 1 miner → belt → matching `Iron` output.
**Script:** run 60 s pinned-active (Fine); separately run 60 s with the cell forced
through promote/demote every 5 s.
**Check:** cumulative `Iron` exported matches within ε (axis A); material exact (axis
C); the belt never carries non-`Iron` (axis D). Verifies `F01 S05/S06`, `F05 S03/S04`,
`F06 S01/S02`.

## S-SMELT — Micro assembler chain
**Setup:** `Iron` cell: miner → belt → assembler(Smelt Iron Plate) → belt →
`IronPlate` output.
**Script:** 60 s Fine, then 60 s with promote/demote cycling.
**Check:** output rate equals the bottleneck (miner vs assembler `craftTime`) within ε;
inputs/outputs satisfy the recipe ratio exactly (conservation); P-LOD holds.
Verifies `F06 S04`, `F05 S04`, `F01 S06`.

## S-MIX — Single-resource enforcement (adversarial)
**Setup:** a belt fed by two sources of different resources at a merge.
**Check:** the belt never simultaneously holds two resource types; the second resource
is back-pressured/refused, never merged — `F06-A1`, `F06 S03`, axis D.

## S-ROUTE — Cross-cell routing & back-pressure
**Setup:** `Iron` mining cell → road → consumer cell (or sink); add a second branch to
force a split.
**Script:** run with ample supply, then block the downstream for a window, then
unblock.
**Check:** delivered throughput = road `rate` within ε when unblocked; during the block,
roads/outputs fill and the source stalls with **zero loss**; splits divide
deterministically — `F07 S02/S03/S04`, axis C/B.

## S-CHAIN — Multi-stage macro chain
**Setup:** `Iron` mine → road → Manufacturer(Smelt Iron Plate) → road →
Manufacturer(next-tier recipe).
**Script:** 120 s; introduce a downstream stall mid-run.
**Check:** steady-state output = slowest stage within ε; back-pressure reaches the
source; total material conserved across the chain — `F08 S02/S03/S04`, axis A/C.

## S-BLUEPRINT — Capture & replicate
**Setup:** build the S-SMELT layout in cell A; `capture(A)`; `paste` into compatible
cell B; reject-paste into an incompatible cell C.
**Check:** B reproduces A's layout exactly and, once steady, yields the same profile
(`F09-A1/A3`); the C paste applies nothing and reports why (`F09-A2`).

## S-DETERMINISM — Full mixed session
**Setup:** a scripted session touching mining, routing, manufacturing, and a
blueprint, with the camera entering/leaving cells (driving LOD).
**Script:** the same `(tickNo, command)` log run twice (and on two builds).
**Check:** dumps at every checkpoint are byte-equal — `F01-A1`, axis B. This is the
end-to-end determinism gate.

## S-PERF (informational, Post-scoring)
**Setup:** a large world (e.g. 256×256) with many developed-but-inactive cells and one
active cell.
**Check:** per-tick cost scales with the active set + O(1)·K coarse cells, not with
total micro entities — `F01-A3`. Reported as a profile, not a pass/fail.

---

### Tolerances (recap)
- ε: cumulative export over `T = 60 s` within ±1 unit/resource or ≤1%, whichever
  larger (`03-simulation-lod §7`).
- Material conservation: **exact**.
- Determinism: **byte-equal** dumps.
