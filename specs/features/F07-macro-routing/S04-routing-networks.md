# F07 · S04 — Multi-Cell Routing Networks & Deterministic Splitting

**Feature:** F07 — Macro Routing: Roads & Big Belts
**Scope:** macro · **MVP:** yes · **Depends on:** S02, S03

## Intent
Compose roads into networks that fan in and fan out across many cells, with
deterministic splitting/merging so large factory graphs behave reproducibly.

## Behavior
1. Roads connect into chains and junctions. A cell/road feeding multiple downstreams
   **splits** its available units among them in a documented deterministic rule (see 
   below). A junction fed by multiple upstreams **merges** by a documented order.
2. Splitting respects each downstream's capacity: a full downstream is skipped that
   tick and its share offered to the next, deterministically.
3. The whole network conserves material end-to-end and reaches a deterministic
   steady-state distribution governed by the bottleneck stage.

### Splitting rule (one source → many downstreams)
When a source (a `MacroOutput.buffer` or `RoadSegment`) has multiple outbound 
destinations (neighboring cells or roads):

1. **Sort destinations** by ascending `(x, y)` coordinate, with `x` as the primary key.
2. **Each tick**, attempt to send units in that sorted order, respecting each 
   destination's capacity:
   - Offer the first destination up to `available / num_downstreams` units.
   - Any units refused (due to full buffer) are offered to the next destination.
   - Each destination gets one "turn" per tick; if it refuses, its quota does not 
     roll to others that same tick (it will be retried next tick).
3. **Tie-breaker:** if two destinations have identical `(x, y)` (impossible in a 
   normal grid but handle for completeness), use the ascending entity id of the 
   destination.

**Why:** Fixed order ensures consistent delivery patterns across runs. The 
per-destination quota prevents a single hungry sink from starving others.

## Data touched
Multiple `RoadSegment`s, `MacroOutput.buffer`s, manufacturer inputs.

## Acceptance criteria
- **AC-1** *Given* a one-source two-sink split, *then* units divide per the documented
  rule, identically across runs.
- **AC-2** *Given* one full sink, *then* its share is redirected/held deterministically
  with no loss.
- **AC-3** *Given* a fixed network and fixed inputs, *then* per-cell delivered totals
  are identical every run (F07-A4) and conserve material (F07-A3).

## Out of scope
Pathfinding / auto-routing (player lays roads explicitly); manufacturing (F08).
