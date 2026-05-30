# F03 · S01 — Deterministic World Generation from Seed

**Feature:** F03 — World Map & Grid
**Scope:** macro · **MVP:** yes · **Depends on:** F02 S01

## Intent
Build the macro grid from a `WorldConfig`: place ore terrains among empty cells using
only the seed, so the same config always yields the same map.

## Behavior
1. `generate(WorldConfig)` produces a `mapWidth × mapHeight` grid; every coord gets a
   `Cell` with a `Terrain`, `Occupancy = Wild`, and (for resource cells) an
   `OreDeposit`.
2. All placement randomness derives from `WorldConfig.seed` via a documented,
   reproducible procedure (CONSTITUTION Art. IV). No wall-clock or unseeded source.
3. Resource distribution yields a playable mix of `Stone/Iron/Copper` among `Empty`
   cells (clusters acceptable); exact algorithm is the implementor's, but it must be
   seed-deterministic.
4. `cellSize` is recorded and fixed for the world's lifetime.

## Data touched
`World`, `Cell`, `Terrain`, `OreDeposit`, `Occupancy`.

## Acceptance criteria
- **AC-1** *Given* the same `WorldConfig`, *then* two generations produce identical
  terrain and deposits at every coord (F03-A1).
- **AC-2** *Given* different seeds, *then* maps differ (the seed actually drives
  placement).
- **AC-3** *Then* every cell is in-bounds, has exactly one terrain, and resource cells
  have a deposit while `Empty` cells do not.

## Out of scope
Outputs (S04); occupancy transitions (S03); depletion dynamics (S02).
