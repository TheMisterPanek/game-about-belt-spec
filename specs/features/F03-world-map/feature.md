# F03 — World Map & Grid

**Module:** `world` · **Scope:** macro · **MVP:** yes

## Intent

The macro playfield: a finite `mapWidth × mapHeight` grid of `Cell`s, each with a
`Terrain`, an `Occupancy`, and (once developed) an interior and a profile.
Deterministic generation from the world `Seed` places ore deposits among empty cells.
This feature owns the grid, terrain, deposits, occupancy transitions, and the fixed
per-cell `Output` set that the micro factory must be planned around.

## Scope

In: world generation, grid storage & coordinate rules, terrain/deposit model,
occupancy state machine (`Wild → Mine/Road/Manufacturer`), per-cell output
generation, neighbor queries. Out: what happens *inside* a Mine (F05/F06), routing
between cells (F07), macro manufacturers' behavior (F08).

## Data

`WorldConfig`, `World`, `Cell`, `Terrain`, `OreDeposit`, `Occupancy`, `MacroOutput`
(`02-data-model.md` §1–2).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Deterministic world generation from seed | ✅ |
| S02 | Cell terrain, ore deposits & depletion | ✅ |
| S03 | Occupancy state machine & transitions | ✅ |
| S04 | Per-cell fixed outputs | ✅ |
| S05 | Grid coordinates & neighbor queries | ✅ |

## Feature-level acceptance

- **F03-A1** Two generations with the same `WorldConfig` produce identical grids
  (same terrain and deposits at every coord).
- **F03-A2** Occupancy only follows legal transitions (S03); illegal commands are
  rejected without mutating state.
- **F03-A3** Every resource cell exposes a non-empty, fixed `outputs` list at
  generation that does not change for the world's lifetime (Invariant: outputs are
  the planning constraint).
