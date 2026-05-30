# F06 · S03 — Belt Junctions & Deterministic Hand-off

**Feature:** F06 — Micro Logistics: Miners, Belts, Power
**Scope:** micro · **MVP:** yes · **Depends on:** S02

## Intent
Define how items pass between adjacent belts (and between belts and machines) where
multiple inputs or outputs meet, deterministically, so two implementations agree
item-for-item.

## Behavior
1. When a belt's output edge meets another belt's input edge in a compatible
   direction, items hand off one at a time, respecting downstream capacity and the
   single-resource rule (a hand-off that would mix resources is refused, I1).
2. **Merges (many→one):** when multiple upstream belts feed one downstream, the
   downstream accepts in a documented priority order (e.g. by ascending source tile
   coord, or round-robin keyed by tile order) — fixed and deterministic.
3. **Splits (one→many):** when one belt feeds multiple downstreams, output is offered
   in a documented order (same keying) so distribution is reproducible.
4. Hand-off respects back-pressure: a full downstream is skipped that tick.

## Data touched
`BeltState` of adjacent belts; machine input edges (`Miner`/`Assembler`).

## Acceptance criteria
- **AC-1** *Given* a many→one merge with all inputs supplied, *then* items are accepted
  in the documented deterministic order, identical across runs.
- **AC-2** *Given* a one→many split, *then* distribution follows the documented order
  deterministically.
- **AC-3** *Given* a hand-off that would mix two resources, *then* it is refused
  (Invariant I1 preserved).
- **AC-4** *Given* a full downstream, *then* it is skipped that tick with no loss.

## Out of scope
Belt advancement within a single belt (S02); assembler internals (S04).
