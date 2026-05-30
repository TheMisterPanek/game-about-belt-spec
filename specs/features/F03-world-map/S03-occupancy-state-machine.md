# F03 · S03 — Occupancy State Machine & Transitions

**Feature:** F03 — World Map & Grid
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Govern how a cell changes role — from untouched to mine, road, or manufacturer — with
legal transitions only, so the macro map stays consistent.

## Behavior
1. `Occupancy ∈ {Wild, Mine, Road, Manufacturer}`. New cells are `Wild`.
2. Legal transitions:
   - `Wild → Mine` (develop a cell with a micro grid; requires resource terrain for
     useful mining, but any cell may be developed).
   - `Wild → Road` (only on `Empty` terrain; F07).
   - `Wild → Manufacturer` (only on `Empty` terrain; F08).
   - `Road → Wild`, `Manufacturer → Wild`, `Mine → Wild` (tear down; only when empty
     of in-flight material, else rejected).
3. Invariant I3: `Occupancy == Mine` **iff** the cell has a `micro` grid; Roads and
   Manufacturers have none.
4. Illegal transitions are rejected without mutating state (F03-A2).

## Data touched
`Cell.occupancy`, `Cell.micro`.

## Acceptance criteria
- **AC-1** *Given* a `Wild` `Empty` cell, *then* `→Road`, `→Manufacturer`, `→Mine` are
  all legal; *given* a resource cell, *then* `→Road`/`→Manufacturer` are rejected.
- **AC-2** *When* a cell becomes `Mine`, *then* it gains a micro grid; *when* it leaves
  `Mine`, *then* its micro grid is removed (Invariant I3).
- **AC-3** *Given* a tear-down while material is in flight, *then* it is rejected until
  drained.
- **AC-4** *Given* any illegal transition command, *then* state is unchanged.

## Out of scope
Road/manufacturer behavior (F07/F08); micro contents (F05/F06).
