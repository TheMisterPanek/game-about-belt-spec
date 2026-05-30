# Spec Improvements: Closing 12 Decision-Point Gaps

**Date:** 2026-05-30  
**Purpose:** Make the specification precise enough for a 4b model to implement without architectural decisions.

## Summary

This audit identified 12 decision-point gaps where the spec delegated implementation choices (suitable for a senior engineer) but ambiguous for a model with no domain context. Each gap has been closed by adding explicit constraints, formulas, or rationale to the spec.

## Changes by Gap

### High-priority (LOD correctness & determinism)

| Gap | Issue | File | Fix |
|-----|-------|------|-----|
| 1 | LOD warm-up duration unclear | `F01 S04` | Add K=10 ticks rule + variance threshold (±1 unit/s) |
| 3 | Coarse starvation scaling formula missing | `F01 S05` | Add `scale = A/D` formula with operation order |
| 6 | Profile stability window undefined | `F05 S04` | Add K=20 ticks + max rate change threshold |
| 7 | Dirty flag lifecycle ambiguous | `F05 S04` | Add when set, preserved, cleared with promotion/demotion rules |
| 10 | Entity ID assignment unspecified | `F01 S02` | Add monotonic global ID rule; no reuse on promotion |
| 11 | Fixed-point operation order partial | `F01 S03` | Add coarse scaling, demotion rounding, multiply-before-divide rule |

### Medium-priority (determinism & correctness)

| Gap | Issue | File | Fix |
|-----|-------|------|-----|
| 4 | Routing split order ambiguous | `F07 S04` | Add sort-by-(x,y), per-destination quota rule |
| 5 | Belt junction merge priority undecided | `F06 S03` | Add ascending-coord sort for both merge and split |
| 8 | Belt advancement order unspecified | `F06 S02` | Add ID-order iteration rule with back-pressure logic |

### Lower-priority (edge cases)

| Gap | Issue | File | Fix |
|-----|-------|------|-----|
| 2 | Profile seeding mechanism vague | `F01 S06` | Add ±20% goal, belt/assembler seeding strategy, verification |
| 9 | Inventory folding method undefined | `F01 S06` | Add 1:1 item/unit equivalence, per-belt/assembler fold method |
| 12 | Occupancy/micro coupling unclear | `F03 S03` | Add lifecycle: creation on develop, deletion on occupancy change, preservation on demotion |

---

## Files Modified

1. **F01 S02** (ECS Schedule): Entity ID assignment rule
2. **F01 S03** (Fixed-Point Clock): Operation order for coarse scaling, rounding carry, multiply-before-divide
3. **F01 S04** (LOD Scheduler): Warm-up termination rule (K=10 ticks, ±1 unit/s threshold)
4. **F01 S05** (Coarse Production): Starvation scaling formula
5. **F01 S06** (Promotion/Demotion): Seeding strategy (±20% goal, per-entity seeding), inventory folding rule
6. **F03 S03** (Occupancy State Machine): Micro grid lifecycle rules
7. **F05 S04** (Profile Computation): Stability detection (K=20 ticks), Dirty flag lifecycle
8. **F06 S02** (Belt Transport): Belt advancement order (ascending entity ID)
9. **F06 S03** (Belt Junctions): Merge priority (ascending-coord sort) and split distribution rule
10. **F07 S04** (Routing Networks): Splitting rule (ascending-coord sort, per-destination quota)

---

## Key Principles Added

1. **Deterministic ordering:** All multi-choice decisions (splits, merges, advancement order) use 
   ascending coordinate or entity ID, applied consistently.

2. **Explicit thresholds:** Qualitative terms ("stabilizes," "proportionally") replaced with 
   concrete numbers: 10 ticks warm-up, 20-tick stability window, 1 unit/s threshold, ±20% seeding tolerance.

3. **Operation order:** Fixed-point arithmetic order specified (multiply-before-divide, carry 
   remainders forward) to ensure bit-for-bit cross-language equivalence.

4. **Lifecycle clarity:** State creation/deletion rules for micro grids, entity IDs, and dirty flags 
   are now explicit.

5. **Rationale notes:** Each rule includes a "Why:" paragraph explaining the constraint that forces 
   the design (e.g., "ID-order advancement prevents item collision").

---

## Testing Implications

These changes close gaps that would have caused test failures:

- **P-LOD property:** Gaps 1, 3, 6, 7 were blocking; now the warm-up, starvation, and profile 
  logic are deterministic.
- **Determinism tests:** Gaps 4, 5, 8, 10, 11 were causing divergence between runs; now split/merge/
  advancement orders are fixed.
- **Golden-master tests:** Rounding and folding (gap 11, 9) are now specified; cross-language 
  differential tests should now pass.

---

## Next Steps for Implementation

When a 4b model reads the spec, it will find:
- No degree of freedom in warm-up duration, sorting order, or operation precedence.
- Explicit formulas for starvation scaling and profile updates.
- Clear lifecycle rules for state creation/deletion.

The model should be able to implement each story without architectural decisions — only converting 
spec rules to code ("how") rather than inferring what to implement ("what").
