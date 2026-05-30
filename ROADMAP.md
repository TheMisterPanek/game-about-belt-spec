# ROADMAP

How to **build** this game from the specs: an ordered set of **vertical-slice
milestones**, each one a thin end-to-end increment that is independently
*demonstrable* and *benchmark-scorable* (via the headless harness, `F01 S07`). The
same slicing applies to every implementation (any language/engine) so benchmark runs
stay comparable.

This file is the *shared* build plan. Per-run status lives in `PROGRESS.md`. "Done"
vs "Verified" is defined in `specs/README.md` → **Definition of Done**.

## How to iterate

1. Take the lowest-numbered milestone that is not `Verified`.
2. Implement exactly its stories — no more (resist building ahead; later slices depend
   on earlier ones being correct, not large).
3. Run the milestone's **exit scenarios** on the headless harness and check its
   **exit criteria**.
4. Update `PROGRESS.md`: stories → `Done` when their ACs pass, `Verified` when the
   milestone's benchmark scenarios pass.
5. Only advance when every exit criterion is met. A red exit criterion blocks the next
   slice — fix it before widening scope.

Each milestone leaves the project in a **scorable** state. You should never have a long
stretch with nothing to measure.

## Critical path (why this order)

```
M0 catalog+world+clock+harness   ──► everything (data + determinism substrate)
M1 one cell mines & exports (Fine) ─► gives LOD something real to reduce
M2 LOD core (profile/coarse/promote/demote)  ◄── THE signature milestone (P-LOD)
M3 micro assembler + power        ──► richer cells to test LOD against
M4 macro routing (roads)          ──► move products between cells
M5 macro manufacturing + chains   ──► transform across cells (needs M4)
M6 blueprints                     ──► replicate M1–M5 layouts
M7 interaction shell (playable)   ──► WASD/mouse; enter/leave drives LOD (needs M2)
M8 persistence (Post-MVP)         ──► save/load via snapshot or replay
```

The headless harness (M0) means M1–M6 are fully scorable **without** the interaction
shell; F04 is deliberately late (M7) because scoring never needs a GUI. The
`pinActive`/`unpinActive` commands F04 uses already exist and are tested headlessly in
M2.

---

## M0 — Walking skeleton & harness
**Goal:** a deterministic world you can generate, tick, script, and dump — with nothing
moving yet.
**Stories:** F02 S01–S03 · F03 S01, S02, S05 · F01 S01, S02, S03, S07.
**Demo:** generate `{seed:42, 64×64}`, dump the grid, tick 1000×, dump again.
**Exit criteria:**
- Gates G1–G3 pass (`benchmark/acceptance.md`).
- **S-GEN** passes (identical grids for identical config; different seed ⇒ different
  grid).
- A no-op 1000-tick run is byte-identical across two runs (determinism, axis B).

## M1 — One cell mines & exports (fine-grained only)
**Goal:** develop a single ore cell, build miner→belt→output, watch it fill the cell's
macro output buffer. Cell is pinned active; **no LOD yet**.
**Stories:** F03 S03, S04 · F05 S01, S02, S03 · F06 S01, S02, S03.
**Demo:** S-MINE layout, pinned active, 60 s → macro `Iron` buffer rises at miner rate.
**Exit criteria:**
- **S-MINE** (all-Fine portion) and **S-MIX** pass.
- Conservation exact (axis C); single-resource invariant I1 holds (axis D).

## M2 — LOD core *(signature milestone)*
**Goal:** the two-tier simulation. A cell reduces to a `ProductionProfile`; inactive
cells advance by `rate × Δt`; entering/leaving promotes/demotes **losslessly**.
**Stories:** F05 S04 · F01 S04, S05, S06.
**Demo:** run S-MINE all-Fine vs forced promote/demote every 5 s; totals match.
**Exit criteria:**
- **P-LOD** passes for S-MINE: coarse vs fine cumulative export within ε; material
  exact across transitions (axis A — the highest-weighted axis).
- Repeated promote/demote cycles show no drift.

## M3 — Micro assembler & power
**Goal:** craft inside a cell; gate machines on power.
**Stories:** F06 S04, S05.
**Demo:** S-SMELT layout (miner→assembler→output), Fine and LOD.
**Exit criteria:**
- **S-SMELT** passes both all-Fine and with promote/demote (P-LOD holds for a crafting
  cell).
- Recipe conservation exact; cutting power halts consumers the same tick.

## M4 — Macro routing (roads / big belts)
**Goal:** move one resource between cells over roads, with throughput limits and
back-pressure.
**Stories:** F07 S01, S02, S03, S04.
**Demo:** S-ROUTE — mine cell → road → sink; block/unblock downstream.
**Exit criteria:**
- **S-ROUTE** passes: throughput = road rate within ε; block causes lossless
  back-pressure; splits deterministic; single-resource I1 on roads.

## M5 — Macro manufacturing & chains
**Goal:** transform resources at macro scale and compose multi-stage cross-cell chains.
**Stories:** F08 S01, S02, S03, S04.
**Demo:** S-CHAIN — mine → road → manufacturer → road → manufacturer.
**Exit criteria:**
- **S-CHAIN** passes: steady-state = bottleneck within ε; back-pressure propagates;
  material conserved across the chain.

## M6 — Blueprints
**Goal:** capture a working cell and replicate it; atomic paste.
**Stories:** F09 S01, S02, S03, S04.
**Demo:** S-BLUEPRINT — capture S-SMELT cell, paste to a compatible cell, reject on an
incompatible one.
**Exit criteria:**
- **S-BLUEPRINT** passes: paste reproduces layout exactly and yields the same profile
  at steady state; invalid paste applies nothing (atomic).

## M7 — Interaction shell (playable)
**Goal:** make it a game: MapView↔CellView, WASD camera, mouse placement, enter/leave
drives LOD.
**Stories:** F04 S01, S02, S03, S04.
**Demo:** play — pan the map, enter a cell (it goes Fine), build, leave (it goes
Coarse).
**Exit criteria:**
- All F04 story ACs pass; camera changes never alter logical state (F04-A3).
- **S-DETERMINISM** (full mixed session, incl. enter/leave) passes byte-equal.
- **MVP COMPLETE** at the end of M7.

## M8 — Persistence *(Post-MVP)*
**Goal:** save and reload exactly.
**Stories:** F10 S01, S02, S03, S04.
**Exit criteria:**
- **S** save→load is logically equal at the saved `tickNo` (snapshot and/or replay);
  transient micro entities reconstructed via promotion; material conserved.

---

## Milestone → benchmark map (quick reference)

| Milestone | Primary scenarios | Dominant axes |
|-----------|-------------------|---------------|
| M0 | S-GEN | B (determinism), gates |
| M1 | S-MINE (Fine), S-MIX | C, D |
| M2 | S-MINE (LOD) | **A (P-LOD)** |
| M3 | S-SMELT | A, C |
| M4 | S-ROUTE | C, D, B |
| M5 | S-CHAIN | A, C |
| M6 | S-BLUEPRINT | G |
| M7 | S-DETERMINISM | B |
| M8 | (persistence round-trips) | bonus |
