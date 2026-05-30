# PROGRESS

Per-implementation status tracker. **Copy this file per benchmark run** (e.g.
`PROGRESS.<language>.md`) and update it as you build — the spec tree itself stays
unchanged. Status meanings come from `specs/README.md` → **Definition of Done**.

**Status legend:** `☐` not started · `◐` in progress · `☑` Done (ACs pass headless) ·
`✅` Verified (milestone's benchmark scenarios pass)

> Implementation: `<name / language / engine>`
> Date started: `<yyyy-mm-dd>` · Target tier (Bronze/Silver/Gold): `<…>`

## Milestones

| Milestone | Status | Exit scenario(s) | Notes |
|-----------|:------:|------------------|-------|
| M0 Walking skeleton & harness | ☐ | S-GEN | |
| M1 One cell mines & exports (Fine) | ☐ | S-MINE (Fine), S-MIX | |
| M2 LOD core *(signature)* | ☐ | S-MINE (LOD / P-LOD) | |
| M3 Micro assembler & power | ☐ | S-SMELT | |
| M4 Macro routing | ☐ | S-ROUTE | |
| M5 Macro manufacturing & chains | ☐ | S-CHAIN | |
| M6 Blueprints | ☐ | S-BLUEPRINT | |
| M7 Interaction shell — **MVP done** | ☐ | S-DETERMINISM | |
| M8 Persistence *(Post-MVP)* | ☐ | round-trips | |

## Stories (by feature)

### F01 — Simulation Core & ECS  · _M0, M2_
- ☐ S01 Deterministic fixed-timestep tick loop _(M0)_
- ☐ S02 ECS world, components & ordered schedule _(M0)_
- ☐ S03 Fixed-point clock & rounding contract _(M0)_
- ☐ S04 LOD scheduler & active-set selection _(M2)_
- ☐ S05 Coarse production from profiles _(M2)_
- ☐ S06 Promotion & demotion (lossless) _(M2)_
- ☐ S07 Command & replay seam (headless) _(M0)_

### F02 — Resource & Recipe Catalog  · _M0_
- ☐ S01 Resource type & tier definitions
- ☐ S02 Recipe definitions & validation
- ☐ S03 Catalog queries

### F03 — World Map & Grid  · _M0, M1_
- ☐ S01 Deterministic world generation _(M0)_
- ☐ S02 Terrain, deposits & depletion _(M0)_
- ☐ S03 Occupancy state machine _(M1)_
- ☐ S04 Per-cell fixed outputs _(M1)_
- ☐ S05 Coords & neighbor queries _(M0)_

### F04 — Game States, Camera & Input  · _M7_
- ☐ S01 State machine: MapView ↔ CellView
- ☐ S02 WASD camera pan & zoom
- ☐ S03 Mouse selection & placement → commands
- ☐ S04 Enter/leave drives LOD promotion/demotion

### F05 — Cell View: Finite Factory Floor  · _M1, M2_
- ☐ S01 Micro grid creation & bounded sizing _(M1)_
- ☐ S02 Tile placement & removal validation _(M1)_
- ☐ S03 Output ports & export to macro cell _(M1)_
- ☐ S04 Profile computation from live layout _(M2)_

### F06 — Micro Logistics: Miners, Belts, Power  · _M1, M3_
- ☐ S01 Miner extraction & ore depletion _(M1)_
- ☐ S02 Single-resource belt transport _(M1)_
- ☐ S03 Belt junctions & deterministic hand-off _(M1)_
- ☐ S04 Assembler crafting from recipe _(M3)_
- ☐ S05 Power network: generation, distribution, gating _(M3)_

### F07 — Macro Routing: Roads & Big Belts  · _M4_
- ☐ S01 Lay & clear roads
- ☐ S02 Single-resource directional flow
- ☐ S03 Throughput limits & back-pressure
- ☐ S04 Multi-cell networks & deterministic splitting

### F08 — Macro Manufacturing  · _M5_
- ☐ S01 Place & remove manufacturer
- ☐ S02 Recipe processing with input buffers
- ☐ S03 Integration with routing (demand & supply)
- ☐ S04 Multi-stage cross-cell production chains

### F09 — Blueprints: Copy / Paste  · _M6_
- ☐ S01 Capture a cell layout
- ☐ S02 Validate paste target
- ☐ S03 Atomic paste as batched commands
- ☐ S04 Replicate macro role (road/manufacturer)

### F10 — Persistence & Save Format  · _M8 (Post-MVP)_
- ☐ S01 Versioned save format
- ☐ S02 Snapshot save/load round-trip
- ☐ S03 Replay-based save/load round-trip
- ☐ S04 Load-time reconstruction of transient state

## Benchmark axes (final score)

| Axis | Weight | Status | Score/Notes |
|------|:------:|:------:|-------------|
| A — LOD losslessness (P-LOD) | 30% | ☐ | |
| B — Determinism | 20% | ☐ | |
| C — Conservation | 15% | ☐ | |
| D — Single-resource (I1) | 10% | ☐ | |
| E — Micro feature correctness | 10% | ☐ | |
| F — Macro feature correctness | 10% | ☐ | |
| G — Blueprints | 5% | ☐ | |

_Gates (must pass to score): G1 headless harness · G2 no reals in dumps · G3 catalog
integrity._
