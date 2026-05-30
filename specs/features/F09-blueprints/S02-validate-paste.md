# F09 · S02 — Validate Paste Target

**Feature:** F09 — Blueprints: Copy / Paste
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
Decide whether a blueprint can be pasted onto a given cell before applying anything, so
pastes are predictable and atomic.

## Behavior
1. `canPaste(Blueprint, cell)` returns success only if: target `MicroGrid.size ==
   Blueprint.size`; every building's target tile is in-bounds, empty, and not an output
   tile; and kind-specific constraints hold (e.g. a `Miner` requires ore terrain).
2. It does not mutate state; it returns a result with the first blocking reason if any.
3. Terrain/output compatibility is checked: pasting a layout that relies on outputs the
   target lacks is reported (the paste may still place non-output-dependent buildings,
   but the player is warned per the documented policy).

## Data touched
Reads `Blueprint`, target `MicroGrid`, `Cell` (no mutation).

## Acceptance criteria
- **AC-1** *Given* a size mismatch, *then* `canPaste` fails with a size reason.
- **AC-2** *Given* an occupied/out-of-bounds/output target tile for any building, *then*
  `canPaste` fails citing that conflict.
- **AC-3** *Given* a `Miner` over non-ore, *then* `canPaste` reports the terrain
  conflict.
- **AC-4** *Then* `canPaste` never mutates state.

## Out of scope
Applying the paste (S03); capture (S01).
