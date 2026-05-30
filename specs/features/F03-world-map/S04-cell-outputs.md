# F03 · S04 — Per-Cell Fixed Outputs

**Feature:** F03 — World Map & Grid
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Give each cell a small, fixed set of `MacroOutput`s decided at generation. These are
the planning constraint the micro factory must be designed around (CONSTITUTION
Art. V; the design's core challenge).

## Behavior
1. At generation, each cell receives a fixed, non-empty `outputs` list of
   `MacroOutput { resource, buffer=0, capacity }`. The count is small (e.g. 1–4) and
   their positions correspond to boundary `OutputPort`s on the micro grid (F05 S03).
2. The output set never changes for the world's lifetime; the player cannot add, move,
   or remove outputs.
3. Each output's `resource` is drawn from the catalog and is what that boundary can
   export; a micro factory must route the right resource to the right output.
4. Outputs hold a bounded `buffer` (`0 ≤ buffer ≤ capacity`, Invariant I2) consumed by
   roads (F07) or coarse production (F01 S05).

## Data touched
`Cell.outputs`, `MacroOutput`.

## Acceptance criteria
- **AC-1** *Then* every cell has a fixed, non-empty outputs list that is identical
  across regenerations with the same seed and never changes at runtime (F03-A3).
- **AC-2** *Given* a micro factory, *then* only items matching an output's `resource`
  reaching that output are exported (F05 S03 enforces).
- **AC-3** *Then* each output's buffer stays within `[0, capacity]` at all times.

## Out of scope
Micro output ports/export mechanics (F05 S03); routing pickup (F07).
