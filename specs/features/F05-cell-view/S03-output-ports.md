# F05 · S03 — Output Ports & Export to Macro Cell

**Feature:** F05 — Cell View: Finite Factory Floor
**Scope:** micro · **MVP:** yes · **Depends on:** S01, F03 S04

## Intent
The boundary where the micro factory hands product to the macro layer. Items that
reach an output of the matching resource become the cell's macro output — the literal
micro→macro coupling (CONSTITUTION Art. V).

## Behavior
1. Each `OutputPort` has a fixed boundary `pos`, a `resource`, and a link `macroOut`
   to one of the cell's `MacroOutput`s (F03 S04). Ports are placed at grid creation and
   never moved by the player.
2. `OutputSystem` (schedule step 9): an item arriving at a port whose `resource`
   matches the item is removed from the floor and added to the linked
   `MacroOutput.buffer`, subject to capacity/back-pressure (Invariant I2). A
   mismatched item is **not** exported (it backs up on its belt).
3. When the macro buffer is full, the port stops accepting, back-pressuring the
   upstream belt deterministically.

## Data touched
`OutputPort`, `MacroOutput.buffer`; reads belt/item state at the port.

## Acceptance criteria
- **AC-1** *Given* an item of the port's resource at the port, *then* it is exported
  into the matching `MacroOutput.buffer` (within capacity).
- **AC-2** *Given* an item of a different resource at the port, *then* it is not
  exported and remains on the floor (back-pressure), preserving Invariant I1 upstream.
- **AC-3** *Given* a full macro buffer, *then* the port stops accepting and upstream
  belts back up; no item is lost (Invariant I2).

## Out of scope
Profile computation (S04); belt mechanics (F06); road pickup (F07).
