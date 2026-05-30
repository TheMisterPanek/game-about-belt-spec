# Spec Audit: Decision-Points for 4b Model Implementation

## Executive Summary

The spec is well-written and mostly self-contained, but contains **12 critical decision-point gaps** where the architecture leaves implementation choices to the discretion of a senior engineer. For a 4b model, these gaps are blockers: a model reading only the spec will make plausible-but-wrong choices that compound across systems. The gaps fall into three categories:

1. **Algorithmic details** (deterministic ordering, priority rules, window sizes)
2. **Invariant enforcement** (what triggers what, when state is valid)
3. **Numerical semantics** (rounding in coarse mode, fixed-point order, accumulator semantics)

Each gap is listed with:
- What the spec *says*
- What a 4b model would wrongly *assume*
- Why the spec fails to constrain it
- The fix (one-line rationale or concrete rule)

---

## Gap 1: LOD Scheduler Warm-up Duration

**Location:** `F01 S04` — LOD Scheduler & Active-Set Selection

**What the spec says:**
> "Cells recently edited (`Dirty`) are kept active until their profile stabilizes (output rate variance below a threshold over a window)."

**What a 4b model wrongly assumes:**
- "Stabilize" means wait 1 tick (or 2, or 10?). It picks a number.
- Maybe: "keep Dirty cells active for 30 ticks no matter what."
- Or: "if the profile changes between ticks, reset the stabilization counter."
- Or: "stabilization is per-resource, so if Iron output is stable but Copper varies, keep it active."

**Why the spec fails:**
The words "window," "threshold," and "stabilizes" are qualitative. The spec does not name:
- Window size in ticks (e.g., "measure variance over the last 10 ticks").
- Threshold definition (e.g., "variance < 1 unit/s" or "max/min ratio < 1.1").
- Reset rule (if variance spikes, does the window restart?).

**Impact:** A cell marked Dirty could demote too early (producing wrong output), stay active indefinitely (performance leak), or oscillate promotion/demotion (correctness cascade).

**Fix:**
Add a paragraph to `F01 S04`:
> "Warm-up termination: a Dirty cell remains active until both conditions hold for K=10 consecutive ticks: (1) the profile has not changed since the last observation, (2) the max rate change across any output/demand RateEntry is ≤1 unit/s. On either condition fail, the counter resets. Once both hold for K ticks, the profile is marked valid and Dirty clears."

---

## Gap 2: Profile Seeding Mechanism

**Location:** `F01 S06` — Promotion & Demotion, step 1 (seeding)

**What the spec says:**
> "Seed transient state so the floor is already at the steady state the profile implied — belts pre-filled, assembler buffers/progress primed — so there is no visible 'cold start' pop. The seeding target is the profile's rates."

**What a 4b model wrongly assumes:**
- Fill every belt slot with one item each (wasteful, wrong rate).
- Create items equal to `rate` (but rate is per second; items are discrete).
- Seed assembler progress to 50% of craftTime (arbitrary).
- Seed assembler input buffers to the recipe's input counts (might be 0 to complete one craft immediately).

**Why the spec fails:**
The spec names the *goal* (steady-state output within ε over the next second) but not the *mechanism*. It says "belts pre-filled" and "buffers/progress primed" but not:
- How many items per belt? (Depends on belt speed, item size, rate.)
- What's the formula for assembler seeding? (Input/output ratio? Partial recipes mid-progress?)
- How do you verify the seeding is correct after promotion? (By running 1 tick and checking output matches profile rate within ε.)

**Impact:** Promotion could produce a cold-start dip (bursts of output, then a lag) that violates P-LOD tolerance ε. The 4b model might seed too much (overflow output buffer on first tick) or too little (drop below expected rate).

**Fix:**
Add a detailed section to `F01 S06`:
> "Seeding strategy: the goal is that the cell's first-tick output equals the profile's rate within ±20%. (1) For each Belt with resource R and rate component in the profile: pre-fill slots with items such that the belt will emit `floor(profile_rate × Δt × 0.5)` units in the next tick at steady flow. (2) For each Assembler with recipe producing rate R: set input buffers to half the recipe's input requirements and progress to `craftTime / 2`. (3) After seeding, simulate one tick and verify output within ±20% of profile rate; if not, adjust progress/buffer values and try again. This seeding is deterministic given the profile."

**Note:** This is complex; the exact formula depends on the micro system order and belt physics. A simpler fix: reference a worked example in `benchmark/scenarios.md` showing the seeding for a simple mining + smelting layout.

---

## Gap 3: Coarse Starvation Scaling

**Location:** `F01 S05` — Coarse Production, AC-3

**What the spec says:**
> "If demand cannot be fully met, achievable output scales down proportionally and deterministically (a starved cell produces less)."

**What a 4b model wrongly assumes:**
- Scale factor is `available_input / total_demand`.
- Apply it globally to all outputs? Or per-resource output?
- Apply it to demands or not? (If demand is 10 units and only 5 available, do we pull 5 and scale output to 50%? Or scale demand first?)
- Rounding: after scaling, do fractional outputs get floored and carried like normal, or truncated?

**Why the spec fails:**
The spec says "scales down proportionally" but not:
- The formula (demand_scale = available / demanded?).
- The scope (does demand starvation in one resource starve all outputs, or just consumers of that resource?).
- Rounding semantics (can a cell's profile have two outputs: A=5.5 units/s, B=3.2 units/s; if starved to 50%, does A become 2.75 and B becomes 1.6, then floor to 2 and 1 with carries? Or does the scaling happen after flooring?).

**Impact:** Two implementations could compute different output amounts under starvation. The 4b model might produce too much (forgetting to scale), or scale the wrong dimension (outputs instead of inputs).

**Fix:**
Add to `F01 S05`:
> "Starvation scaling: if the cell's total input demand `D = Σ(demand_rate × Δt)` exceeds available supply `A`, compute `scale = A / D` and reduce achievable output: for each output RateEntry, compute `scaled_rate = output_rate × scale`. Then proceed with coarse production using the scaled rates, flooring with carry as usual (§3). Demands are pulled proportionally: each demanded resource gets `floor(demand_rate × scale × Δt)`, capped by available supply."

---

## Gap 4: Routing Network Splitting Order

**Location:** `F07 S04` — Multi-Cell Routing Networks, AC-1

**What the spec says:**
> "A cell/road feeding multiple downstreams **splits** its available units among them in a documented deterministic rule (e.g. round-robin by ascending destination coord)."

**What a 4b model wrongly assumes:**
- "Ascending destination coord" means sort by `(x, y)` tuple. But which is primary, x or y?
- Round-robin by ascending coord: does that mean sort once at creation and iterate, or sort by coord every tick?
- If a destination is full (can't accept), do we skip it and move to next, or do we mark it "tail of queue" and try again next tick?
- What if the source is a `MacroOutput.buffer` with mixed resources? Is splitting per-resource or global? (Spec says single-resource, so maybe this isn't an issue, but it needs confirming.)

**Why the spec fails:**
The words "round-robin by ascending destination coord" are too vague. There are at least three reasonable interpretations:
1. Sort neighbors by coord (x first). In each tick, try to send to the first unsated neighbor, then second, etc.
2. Maintain a per-source round-robin state: the index of the "next" neighbor to try. Each tick, try that neighbor; on move to next; wrap at end.
3. Track per-neighbor: round-robin per source-destination pair (more complex).

Each produces different results under different load patterns.

**Impact:** A splitter feeding two downstreams will distribute units non-identically across runs, violating `F07-A4` (fixed network → fixed outputs).

**Fix:**
Add to `F07 S04`:
> "Splitting rule: when a source (MacroOutput or RoadSegment) has multiple outbound destinations, sort them by `(x, y)` with x primary. In each tick, attempt to send units in that sorted order, respecting each destination's capacity: offer up to `available / num_downstreams` to the first, any remainder to the second, etc. If a destination is full and accepts 0, its quota rolls over; do not redistribute to other downstreams this tick. (Each downstream gets one 'turn' per tick, deterministically.) Tie-breaker for cells with the same `(x, y)` (impossible in a normal grid, but handle it): use the ascending entity id of the destination."

---

## Gap 5: Belt Junction Priority

**Location:** `F06 S03` — Belt Junctions & Deterministic Hand-off, AC-1

**What the spec says:**
> "Merges (many→one): when multiple upstream belts feed one downstream, the downstream accepts in a documented priority order (e.g. by ascending source tile coord, or round-robin keyed by tile order) — fixed and deterministic."

**What a 4b model wrongly assumes:**
- "Priority order" is fixed at creation (sort neighbors once).
- Or it's per-tick round-robin, and the state persists across ticks.
- Or it's per-tick but resets each tick (start at first neighbor again).
- "Ascending source tile coord" — same ambiguity as Gap 4: which coord is primary?
- If an upstream is blocked (full), do we skip it and try the next, or hang the whole merge?

**Why the spec fails:**
The spec gives two options ("ascending source tile coord, or round-robin") but doesn't pick one. Both are reasonable; a 4b model will pick one and stick with it, but if the intended rule is the other, results diverge.

**Impact:** Two implementations merge items in different orders. Downstream logic that depends on item order (e.g., "if the first item from upstream A is iron, start this recipe") will diverge deterministically.

**Fix:**
Add to `F06 S03`:
> "Merge priority: when multiple upstreams feed a single input, accept items in ascending order of source tile coord `(x, y)`, with x primary. In each tick, poll each upstream in that sorted order until the downstream is full or all have been offered a turn. On collision (two upstreams with the same `(x, y)`): use ascending entity id. Merges are greedy per tick: any upstream that has an item to offer and the downstream has space will hand off one item, then the next upstream gets a turn (with the same space remaining)."

---

## Gap 6: Profile Stability Window Size

**Location:** `F05 S04` — Profile Computation, AC-3

**What the spec says:**
> "'Steady state' is measured over a window; while a cell is Dirty or its rates vary beyond a threshold, the profile is marked `valid = false` and the scheduler keeps the cell active."

**What a 4b model wrongly assumes:**
- Window = last 5 ticks. Or last 30. Or 100.
- Threshold = "rate never changes" (strict).
- Or "rate changes by < 1 unit/s per tick."
- Or "standard deviation of rate < 10%."

**Why the spec fails:**
The spec uses the word "window" but doesn't give a concrete size. It says "varies beyond a threshold" but doesn't define "varies" (delta? ratio? std dev?).

**Impact:** A cell's profile stability toggles at different times, causing unnecessary extra ticks in active simulation, or demoting a cell while its profile is still changing. P-LOD tests fail because the profile is invalid.

**Fix:**
Add to `F05 S04`:
> "Steady-state detection: ProfileUpdateSystem maintains a rolling history of the cell's output rates over the last K=20 ticks. When a cell is marked Dirty, the history clears. Each tick, compute the cell's current output rate (units per second, Fixed). The profile is valid if: (1) the cell's Dirty flag is false, and (2) the max difference between any two rates in the history is ≤ 1 unit/s (absolute tolerance). Once valid for two consecutive ticks, the profile is locked and the cell may demote."

---

## Gap 7: Dirty Flag Semantics

**Location:** `F01 S04`, `F05 S04`, and throughout

**What the spec says:**
> "A Dirty flag (set whenever the micro layout is edited) forces recomputation."

**What a 4b model wrongly assumes:**
- Every command sets Dirty: place, remove, configure assembler.
- Only topology-changing commands: place, remove (but not configure).
- Or: only commands that affect output rate: remove/place building on critical path.
- Does demotion clear Dirty? Does promotion preserve it?
- If a cell is demoted while Dirty, and then promoted again, is it still Dirty?

**Why the spec fails:**
The spec says "whenever the micro layout is edited" but doesn't define "layout." Is a power pole placement a layout edit (it doesn't affect item flow)? Is reconfiguring an assembler recipe a layout edit?

**Impact:** A cell's profile stays invalid forever (if Dirty is never cleared), or is cleared too early (if Dirty is cleared on demotion even though micro state isn't fully synced).

**Fix:**
Add a paragraph to `F05 S04` or `F01 S06`:
> "Dirty flag lifecycle: (1) A cell is marked Dirty when any of the following commands succeed: `place` (any building), `remove` (any building), `configureAssembler` (any recipe change). (2) On promotion, the Dirty flag is preserved. (3) ProfileUpdateSystem clears Dirty when the profile becomes valid (Gap 6). (4) On demotion, Dirty is *not* cleared; the flag is preserved so that if the cell is immediately promoted again, it is aware that the profile may be stale."

---

## Gap 8: Belt Advancement Order

**Location:** `F06 S02` — Belt Transport, AC-4

**What the spec says:**
> "Items advance in a deterministic per-belt order; a belt whose output is blocked back-pressures upstream."

**What a 4b model wrongly assumes:**
- "Per-belt order" means: iterate belts in ID order and advance each.
- Or: advance all North-facing belts, then East, then South, then West.
- Or: iterate by tile coordinate (0,0), (0,1), ..., (cellSize-1, cellSize-1).
- Why does order matter? (Item overlap, vanishing items if one belt blocks before the upstream tries to push.)

**Why the spec fails:**
The spec says "deterministic per-belt order" but not the actual order. It doesn't explain the *why*: if you advance belts in the wrong order, items can pile up in one belt's output buffer, causing items to vanish when the downstream advances and consumes them.

**Impact:** Two implementations advance items in different orders. Under certain layouts, one may back-pressure correctly and one may drop items, or one may stall and one may flow.

**Fix:**
Add to `F06 S02`:
> "Advancement order: BeltSystem iterates belts by entity id in ascending order. For each belt, advance all items in its slots one position toward the output. If the output is blocked (the downstream has no space, or a resource conflict), do not advance and apply back-pressure: the belt is marked blocked and upstream systems (MinerSystem, other BeltSystem pushes) will respect it. This serial order ensures items never overlap or vanish: advancing B1, then B2 → B1's output, means B1 commits space before B2 tries to claim it."

---

## Gap 9: Item/Unit Equivalence in Demotion

**Location:** `F01 S06` — Demotion, step 2 (folding inventory)

**What the spec says:**
> "Conserve in-flight inventory: items currently on belts / in buffers are not destroyed — they are folded into the cell's output buffer or a carried-inventory amount, so total material is conserved."

**What a 4b model wrongly assumes:**
- A belt with 50 items → 50 units of resource (1 item = 1 unit).
- Or: items have no inherent size; count them as-is.
- An assembler with 7 items of Iron in the input buffer → 7 units of Iron?
- An assembler mid-craft with 5/10 inputs and progress = 0.3 craftTime → conserve 5 units of input? Or 0 units (no output yet)?

**Why the spec fails:**
The spec doesn't define the relationship between *items* (discrete entities on belts) and *units* (the quantity in buffers). Are they 1:1? Or does a single item have variable weight?

**Impact:** A cell demotes and loses material (if the model forgets to fold in-flight items), or creates material (if it double-counts), violating material conservation.

**Fix:**
Add to `F01 S06`:
> "In-flight inventory folding: items and units are 1:1 equivalent. (1) For each BeltState on the micro floor: count non-empty slots; add that count to the cell's in-transit buffer (use a temporary accumulator). (2) For each AssemblerState: add its `inputs` map values (total input units accumulated) to the in-transit buffer; the output units (if any) are already in the cell's output buffer. (3) After collecting in-transit total, add it to the appropriate MacroOutput.buffer (by resource) or, if the cell has no matching output, add to a global 'overflow' inventory carried forward if the cell is ever promoted again. Do not lose units."

---

## Gap 10: Entity ID Assignment & Determinism

**Location:** `F01 S02` — ECS Schedule, and everywhere entities are created

**What the spec says:**
> "Within a system, entities are processed in a **deterministic order**: ascending entity id, unless the system spec states another key."

**What a 4b model wrongly assumes:**
- Entity IDs are assigned in creation order (1, 2, 3, ...), globally.
- Or: IDs are per-cell (each cell's micro entities start from 0 or 1).
- Or: IDs are assigned by a hash of position (deterministic but not ascending by creation).
- Does promotion recreate entities with new IDs, or reuse the old IDs from before demotion?

**Why the spec fails:**
The spec mandates *ascending id iteration* but doesn't specify *how IDs are assigned*. This matters because:
- If two systems both iterate entities in ascending ID order, but IDs are assigned non-deterministically, results diverge.
- If a cell is promoted, its micro entities are recreated. Do they get new IDs (after all other IDs) or reuse their old ones?

**Impact:** Two runs of the same scenario produce different entity processing order, causing non-deterministic results (e.g., power routing, miner output order, belt junctions resolve differently).

**Fix:**
Add to `F01 S02`:
> "Entity ID assignment: IDs are 64-bit integers, globally unique per world, assigned monotonically: the first entity gets ID 1, the second 2, etc. IDs never wrap or reuse. When an active cell is demoted, its micro entities are deleted (their IDs are consumed). When the cell is promoted again, its micro entities are created with new IDs (the next available). This ensures that ascending ID iteration is stable and deterministic: if you iterate all micro entities in ascending ID order, you're iterating them in creation order, which is deterministic given the command sequence."

---

## Gap 11: Fixed-Point Operation Order

**Location:** `F01 S03` — Fixed-Point Clock

**What the spec says:**
> "Multiplication/division order is specified where it matters (e.g. `rate × Δt` computed as `Fixed`), so cross-language results match bit-for-bit at the chosen scale."

**What a 4b model wrongly assumes:**
- All operations are associative (they're not; `(a × b) / c ≠ (a / c) × b` in fixed-point).
- Coarse scaling: is `(rate × available) / demanded` or `rate × (available / demanded)`?
- Demotion rounding: when in-flight items are in partial slots, do you round per-belt or per-resource?

**Why the spec fails:**
The spec gives `rate × Δt` as an example but doesn't specify formulas for other operations (starvation scaling, partial item rounding). Without explicit order, two implementations diverge after a few ticks of fixed-point rounding.

**Impact:** Bit-for-bit determinism fails. Cross-implementation tests (differential testing) diverge.

**Fix:**
Add to `F01 S03`:
> "Operation order: (1) Coarse production output: `units = floor((rate × Δt) + remainder)`, where remainder is the accumulated fractional part carried from previous ticks. (2) Coarse starvation scaling: `scaled_rate = (rate × available) / demanded` (multiply first, then divide). (3) Demotion item rounding: for each resource on each belt, count items and add directly; do not round fractional slots. (4) All intermediate computations use Fixed arithmetic (1024-bit denominators, scaled numerators); never convert to Real and back."

---

## Gap 12: Occupancy/Micro Grid Coupling

**Location:** `F03` data model, `F05` feature

**What the spec says (Invariant I3):**
> "`micro` present iff `occupancy == Mine`."

**What a 4b model wrongly assumes:**
- If occupancy is Road or Manufacturer, `micro` is always absent.
- If occupancy is Mine, `micro` is always present (but might be null in the implementation).
- When a player removes the last building from a cell, does `micro` persist (in an empty state) or is it deleted?
- When a cell is demoted, is `micro` deleted only if `occupancy != Mine`?

**Why the spec fails:**
The spec states the invariant but not the *enforcement rule*. It doesn't say when `micro` is created/deleted relative to occupancy changes.

**Impact:** The demotion system gets confused: if it's supposed to demote a Mine cell, it checks for `micro` presence but finds it absent (because someone else deleted it on the last building removal). Or it tries to delete `micro` on a Road, which shouldn't have one.

**Fix:**
Add to `F03 S02` (or `F05 S02`):
> "Micro grid lifecycle: (1) When a cell is first developed (F03 S03), a MicroGrid is created and `micro` is set. (2) When the last building is removed from a Mine cell (via command), `micro` is *not* deleted; it persists as an empty grid. (3) When a cell's occupancy is changed (e.g. from Mine to Road via a command — if such a command exists), `micro` is deleted and set to absent. (4) On demotion (F01 S06), `micro` is deleted; the cell is now coarse. (5) On promotion, `micro` is reconstructed from the stored layout."

---

## Summary of Fixes by Module

| Module | Gap | Spec section | Fix type |
|--------|-----|--------------|----------|
| `sim` | 1 (warm-up) | F01 S04 | Add deterministic termination rule with tick count |
| `sim` | 3 (starvation) | F01 S05 | Add formula for scaled rates and proportional demand reduction |
| `sim` | 4 (split order) | F07 S04 | Specify sorting rule and handling of full downstreams |
| `microfactory` | 2 (seeding) | F01 S06 | Add seeded values formula and verification method |
| `microfactory` | 6 (stability) | F05 S04 | Add window size (20 ticks) and threshold (±1 unit/s) |
| `microfactory` | 7 (Dirty) | F05 S04 | Add lifecycle rules and when flag clears |
| `microfactory` | 9 (demotion folding) | F01 S06 | Add item-count method for in-flight inventory |
| `microfactory` | 12 (I3 coupling) | F03/F05 | Add creation/deletion rules for `micro` field |
| `routing` | 5 (junction) | F06 S03 | Specify priority order and blocking behavior |
| `routing` | 8 (belt order) | F06 S02 | Specify ID-based advancement order and why |
| All | 10 (IDs) | F01 S02 | Specify global monotonic ID assignment and persistence |
| All | 11 (fixed-point) | F01 S03 | Add operation order for coarse scaling and rounding |

---

## Implementation Priority

For a 4b model to proceed without getting stuck:

1. **High priority** (blocks correctness): gaps 1, 3, 6, 7 (LOD scheduler, starvation, stability, Dirty).
2. **High priority** (blocks determinism): gaps 4, 5, 8, 10, 11 (ordering rules for routing, junctions, belts, IDs, fixed-point).
3. **Medium priority** (correctness but less critical): gaps 2, 9, 12 (seeding, folding, occupancy).

A 4b model can implement gaps 2, 9, 12 with reasonable defaults (e.g., "fold items 1:1, preserve Dirty on demotion, delete micro only on demotion"), but will almost certainly get gaps 1, 3–8, 10, 11 wrong without explicit rules.

---

## Testing Implications

Each gap manifests as a test failure:

- **Gap 1 (warm-up):** P-LOD property fails; demoted cells produce wrong rates.
- **Gap 3 (starvation):** S-MIX scenario diverges under under-supply.
- **Gap 4, 5, 8 (ordering):** Differential test diverges; two runs of the same scenario produce different delivery order.
- **Gap 6, 7 (stability, Dirty):** Profile validity inconsistent; cells toggle active/coarse at different times.
- **Gap 10, 11 (IDs, fixed-point):** Determinism test fails; two runs of the same scenario produce byte-different dumps.

A complete test suite will surface all 12 gaps within M2–M3 (the LOD and routing scenarios).
