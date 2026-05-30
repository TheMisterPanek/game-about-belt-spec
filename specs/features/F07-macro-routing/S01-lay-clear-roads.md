# F07 · S01 — Lay & Clear Roads on Empty Cells

**Feature:** F07 — Macro Routing: Roads & Big Belts
**Scope:** macro · **MVP:** yes · **Depends on:** F03 S03

## Intent
Convert empty cells into routing segments and back, the act of building the macro
logistics network.

## Behavior
1. `layRoad(coord, dir, resource)` succeeds only on a `Wild`, `Empty` cell; it sets
   `Occupancy = Road` and creates a `RoadSegment { resource, dir, buffer=0, rate }`.
2. `clearRoad(coord)` removes a road only when its `buffer` is empty (else rejected
   until drained), returning the cell to `Wild/Empty`.
3. A road's `dir` orients flow toward an orthogonally adjacent cell; the resource is
   bound at lay time (changeable only by clearing, Invariant I1).
4. Invalid commands (non-empty terrain, occupied cell, draining road) are rejected with
   a reason, state unchanged.

## Data touched
`Cell.occupancy`, `RoadSegment`.

## Acceptance criteria
- **AC-1** *Given* a `Wild` `Empty` cell, *then* `layRoad` sets it to `Road` with the
  given resource and direction; *given* any other cell, *then* rejected.
- **AC-2** *Given* a road with buffered material, *then* `clearRoad` is rejected until
  it drains.
- **AC-3** *When* a road is cleared (empty), *then* the cell returns to `Wild/Empty`.

## Out of scope
Flow mechanics (S02); throughput (S03); networks (S04).
