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
   downstream accepts in a documented priority order (see below) — fixed and 
   deterministic.
3. **Splits (one→many):** when one belt feeds multiple downstreams, output is offered
   in a documented order (same keying) so distribution is reproducible.
4. Hand-off respects back-pressure: a full downstream is skipped that tick.

### Merge priority (many upstreams → one downstream)
When multiple upstreams feed a single input junction:

1. **Sort upstreams** by ascending `(x, y)` tile coordinate, with `x` as the primary key.
2. **Each tick**, poll each upstream in that sorted order. For each upstream with an 
   item to send and the downstream has space:
   - Hand off one item (respecting single-resource rule: refuse if resource mismatch).
   - Move to the next upstream and continue (with the same space remaining in downstream).
3. **Tie-breaker:** if two upstreams have identical `(x, y)` (impossible in a normal 
   grid but handle for completeness), use ascending entity id of the upstream.

**Why:** Fixed order ensures reproducible item ordering downstream. Greedy per-tick 
hand-off (many items per tick if space allows) models realistic junction behavior 
without artificial per-upstream quotas.

### Split distribution (one upstream → many downstreams)
When one upstream feeds multiple downstreams:
- Use the same sorted-by-coord rule as merges (ascending `(x, y)`, with `x` primary).
- Each tick, offer items in that order; each downstream that accepts one item consumes 
  one "turn."

**Why:** Consistent with merge behavior; downstreams are serviced fairly without 
starvation.

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
