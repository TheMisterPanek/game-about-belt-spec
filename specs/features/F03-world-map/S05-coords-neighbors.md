# F03 · S05 — Grid Coordinates & Neighbor Queries

**Feature:** F03 — World Map & Grid
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Define coordinate rules and adjacency for the macro grid, the basis for routing and
camera navigation.

## Behavior
1. Coords are integer `(x, y)` with `0 ≤ x < mapWidth`, `0 ≤ y < mapHeight`. The grid
   is finite and non-wrapping.
2. `neighbors(coord)` returns the orthogonally adjacent in-bounds cells (4-neighbour),
   in a deterministic order (e.g. N, E, S, W).
3. Out-of-bounds coords resolve to "no cell"; queries never wrap or panic.
4. `cellAt(coord)` and `outputsOf(coord)` are read-only and deterministic.

## Data touched
`World.cells`, `Coord` (read only).

## Acceptance criteria
- **AC-1** *Given* an edge/corner cell, *then* `neighbors` omits out-of-bounds
  directions and is ordered deterministically.
- **AC-2** *Given* an out-of-bounds coord, *then* `cellAt` returns "no cell" without
  error.
- **AC-3** *Then* adjacency is symmetric: if B ∈ neighbors(A) then A ∈ neighbors(B).

## Out of scope
Diagonal movement; pathfinding (not required); road connectivity (F07).
