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

### Micro grid lifecycle (Invariant I3 enforcement)

- **Creation:** When a cell transitions `Wild → Mine` (via command `developCell`), a 
  new `MicroGrid` is created and linked to the cell. The micro grid is populated with 
  empty tiles; it persists for the lifetime of the mine unless explicitly cleared.
- **Deletion on occupancy change:** When a cell transitions out of `Mine` 
  (`Mine → Road`, `Mine → Manufacturer`, or `Mine → Wild`), the `micro` grid is 
  deleted and the cell's `micro` field is set to absent.
- **Deletion on demotion:** When an active `Mine` cell is demoted by the LOD scheduler 
  (F01 S06), the micro entities are deleted; the `micro` grid itself persists (it is 
  not deleted by demotion, only by occupancy change).
- **Persistence through active/coarse cycles:** A `Mine` cell's micro grid layout is 
  never deleted by promotion/demotion; only by occupancy change. This allows the cell 
  to be promoted again without reconstructing the layout from scratch.

**Why:** The micro grid is the persistent *blueprint* of the mine (where buildings 
are); micro entities are the transient *runtime state*. Demotion removes runtime state 
but keeps the blueprint. Occupancy change removes both.

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
