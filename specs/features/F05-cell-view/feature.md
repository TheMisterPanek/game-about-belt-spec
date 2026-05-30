# F05 — Cell View: Finite Factory Floor

**Module:** `microfactory` (grid & outputs) · **Scope:** micro · **MVP:** yes

## Intent

The micro playfield: a bounded `N×N` `MicroGrid` (default 50×50) that is the interior
of one cell — a bounded factory floor, **finite** and constrained by a
small fixed set of boundary **Output**s. The challenge the design wants: plan a
factory *around* its given outputs in limited space. This feature owns the grid,
tile placement rules, the output connectors, and the cell's bridge to the macro layer
(its `ProductionProfile`). The machines that fill it are F06.

## Scope

In: micro grid creation/sizing, tile occupancy & placement validation, boundary
`OutputPort`s and their link to `MacroOutput`s, the `OutputSystem` that exports
finished items, and `ProfileUpdateSystem` (observe steady state → write profile).
Out: miner/belt/assembler/power behavior (F06), copy/paste (F09).

## Data

`MicroGrid`, `Tile`, `OutputPort`, `BuildingKind`, `ProductionProfile`
(`02-data-model.md` §3–4). Systems: `OutputSystem`, `ProfileUpdateSystem`
(`01-ecs-model.md` §4).

## Key constraints

- `MicroGrid.size == WorldConfig.cellSize`, fixed for the world's life.
- A cell's outputs are **fixed at generation** (F03 S04); the player cannot add or
  move them. Factories must route production to the existing outputs.
- A tile holds at most one building (Invariant I3 relationships apply).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Micro grid creation & bounded sizing | ✅ |
| S02 | Tile placement & removal validation | ✅ |
| S03 | Output ports & export to macro cell | ✅ |
| S04 | Profile computation from live layout | ✅ |

## Feature-level acceptance

- **F05-A1** No building may be placed outside `0..size-1` on either axis, nor on an
  occupied tile, nor on an output tile; such commands are rejected.
- **F05-A2** Items reaching an `OutputPort` whose `resource` matches are added to the
  linked `MacroOutput.buffer` (respecting capacity / back-pressure, Invariant I2);
  mismatched items are not exported.
- **F05-A3** After steady state, `profileOf(cell).outputs` equals the cell's measured
  export rate within tolerance ε, and `ProfileUpdateSystem` clears `Dirty`.
