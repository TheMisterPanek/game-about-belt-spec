# Architecture — Testing Strategy

How a conformant implementation **validates itself while it is built**. Like every other
spec here, this is methodology, not tooling: it names *what to test and how to organize
it*, never a test framework or language (CONSTITUTION Art. III). Pick whatever test
runner your target language offers; the strategy below is identical across all of them.

The whole approach rests on one fact established by the architecture: the simulation is a
**deterministic, headless, step-by-step core** (`00-overview.md §2`, `04-module-boundaries.md §4`).
That makes testing unusually decisive — you start a world, step it one tick at a time,
and assert on observable state at every step.

## 1. Principles

1. **Test the core, not the shell.** All meaningful tests run against the headless domain
   core (`catalog` + `world` + `microfactory` + `routing` + `macromfg` + `sim`, plus
   `blueprint`) via the seam of `F01 S07`. Rendering, camera, and input (`interaction`,
   F04) are out of the simulation test scope — only their *isolation* is checked
   (camera/state changes must not alter logical state, `F04-A3`).
2. **Assert on observable state.** Tests read the **state dump** (`benchmark/README.md` →
   *State dump contract*), never internal data structures. A criterion you can only check
   by reading source is wrong (CONSTITUTION Art. IX).
3. **The tick is the unit of validation.** You validate per `tick` (the fixed `1/30 s`
   step), not sub-tick. Invariants are asserted at chosen `tickNo` checkpoints.
4. **Determinism makes tests cheap and total.** Because identical `(config, command
   stream)` ⇒ identical dumps, property and golden-master tests are reliable and a single
   failing checkpoint pinpoints a regression.
5. **No reals in anything asserted.** A real-valued field in a dump is itself a failure
   (gate G2, CONSTITUTION Art. IV).

## 2. The core validation pattern (step-and-validate)

Every level below is a specialization of this loop:

```
world ← generate(WorldConfig)                 # deterministic start
for t in 0..N:
    apply all commands scheduled for tickNo == t     # validated, between ticks
    tick(Δt)                                          # one scheduled pass; tickNo += 1
    if t in checkpoints:
        dump ← stateDump(world)
        assert invariants(dump)                       # §5 catalog
        assert dump == golden[scenario][t]            # §7, when a golden exists
```

A reusable **test harness** wrapping this loop (generate → script → step → dump → assert)
is the single most valuable thing to build first; `F01 S07` requires it to exist anyway.

## 3. Test levels

A pyramid tuned for this project. Unlike typical apps, the upper levels (properties +
golden-master) carry most of the weight here, because determinism makes them exhaustive
and the hard requirements (P-LOD, conservation) are *system* properties, not unit details.

| Level | Scope | What it proves | Primary use |
|------|-------|----------------|-------------|
| **L1 System/unit** | one system over crafted components | a system's local contract & deterministic visit order (`01-ecs-model.md §2`) | fast feedback while writing a system |
| **L2 Slice/integration** | one milestone's modules via commands | modules cooperate through `world` state and the schedule | per-ROADMAP milestone |
| **L3 Property** | invariants over generated inputs | determinism, conservation, I1, non-negativity hold *for all* inputs | the safety net; §5–§6 |
| **L4 Golden-master / differential** | dumps vs stored references / vs another impl | no regression; cross-language equivalence | regression + cross-impl checks; §7 |
| **L5 Scenario/acceptance** | end-to-end scenarios | the benchmark scenarios pass | `Verified` status, §8 |

## 4. The testability seam (recap)

- **Commands** (player intents) are applied between ticks with validation; **systems**
  run during ticks in the fixed schedule (`04-module-boundaries.md §3`). Tests drive the
  core entirely through commands + ticks.
- A session = `WorldConfig` + ordered `(tickNo, command)` events — so a test *is* a small
  command script plus expected dumps. Tests are data, reproducible, and replayable.

## 5. Invariant catalog (assert these)

Properties to assert at checkpoints. Each cites its spec source; most should hold at
**every** tick, P-LOD over an interval.

| Property | Assertion | When | Source |
|----------|-----------|------|--------|
| **Determinism** | two runs of the same `(config, commands)` ⇒ byte-equal dumps | per checkpoint | `F01-A1`, `S01/S07` |
| **Conservation (exact)** | `Σ produced = Σ consumed + buffered + in-flight + exported`, per resource | every tick | F06/F07/F08 ACs |
| **P-LOD (within ε)** | cumulative export over `T=60 s` matches all-Fine reference within ε; material exact | per interval | `F01-A2`, `03 §7` |
| **Single-resource I1** | no `BeltState`/`RoadSegment` lists two resource types | every tick | `F06-A1`, `F07-A2`, I1 |
| **Non-negativity** | no buffer, deposit, or in-flight count goes negative | every tick | I2, F03 S02 |
| **Buffer bounds** | `0 ≤ MacroOutput.buffer ≤ capacity` (back-pressure, no overflow) | every tick | I2 |
| **Catalog integrity** | every `ResourceId`/`RecipeId` resolves; no real values | load + dump | I5, gates G2/G3 |
| **Profile validity** | a coarse cell's `profile.valid == true`; a `Dirty` cell is not coarse | every tick | I4, `F05 S04` |
| **Occupancy/micro coupling** | `micro` present **iff** `occupancy == Mine` | every tick | I3 |
| **Schedule order** | systems ran in `01-ecs-model.md §5` order, once each | per tick | `F01 S02` |

## 6. Property tests & generators

Property tests feed *generated* inputs and assert §5 invariants. Generators are
language-neutral; describe the input space, not the library.

- **Seeds:** a set of world seeds (incl. the fixed `42` reference) → generation
  determinism (S-GEN) and varied layouts.
- **World sizes:** small (for exhaustive checks) and large (for the LOD cost property,
  `F01-A3`).
- **Command scripts:** randomized but valid sequences of `place/remove/layRoad/
  placeManufacturer/paste/pin/unpin`, with `tickNo`s. Invalid commands are *also*
  generated to assert they are rejected without state change.
- **Adversarial inputs:** mixed-resource feeds at merges (S-MIX) to stress I1; full
  buffers to stress back-pressure; starvation to stress proportional scaling
  (`F01 S05` AC-3).
- **Shrinking:** on failure, minimize the command script to the smallest reproducer; since
  sessions are data, shrinking is mechanical.

## 7. Golden-master & differential testing

Determinism turns "did the behavior change?" into a byte comparison.

- **Golden dumps:** for each scenario (`benchmark/scenarios.md`) store the canonical state
  dump at each checkpoint `tickNo` under version control. A test re-runs the scenario and
  diffs against the golden. Any diff is a regression — investigate before updating goldens.
- **Differential (cross-implementation):** because two correct implementations must
  produce identical dumps, run the *same* scenario scripts through two targets (e.g. your
  Rust and Python builds) and diff. Divergence localizes the bug to the first differing
  `(tickNo, field)`. This is the benchmark's core trick, reused as a dev test.
- **Differential (cross-tier):** the **P-LOD recipe** is a differential test against the
  same implementation in two modes — see §9.
- **Rules:** dumps must have stable ordering and contain no reals (else the diff is
  meaningless / G2 fails). Goldens are updated only with a documented reason.

## 8. Determinism harness

The cheapest high-value test:

1. Run a scenario; capture dumps at all checkpoints.
2. Run it again (ideally a second build/machine).
3. Assert byte-equality at every checkpoint.

Wire this around *every* scenario; it catches accidental non-determinism (unordered
iteration, real arithmetic, wall-clock reads) immediately.

## 9. The P-LOD test recipe (the signature)

The highest-weighted property gets a dedicated, reusable procedure (`03-simulation-lod.md §7`):

1. **Reference run:** pin every involved cell active so all run **Fine** for `T = 60 s`;
   record cumulative export per resource.
2. **LOD run:** same scenario, but force the cell(s) through promote/demote cycles (e.g.
   every 5 s) so they alternate Fine/Coarse.
3. **Assert:** per-resource cumulative export of the LOD run equals the reference within
   **ε** (±1 unit or ≤1%, whichever larger); total material **exactly** equal.
4. **No-drift:** repeat many cycles; assert totals do not accumulate error.

Run this for S-MINE (M2), then S-SMELT (M3) and S-CHAIN (M5) as those slices land.

## 10. Test gates per milestone

Each ROADMAP milestone's exit criteria become a required test set. A milestone is
`Verified` only when these pass (`specs/README.md` → Definition of Done).

| Milestone | Required levels | Must-pass properties / scenarios |
|-----------|-----------------|----------------------------------|
| M0 | L1, L2, L4, L8(det) | gates G1–G3; **S-GEN**; determinism on a no-op run |
| M1 | L1–L3, L5 | conservation, I1; **S-MINE (Fine)**, **S-MIX** |
| M2 | L3, L4, L5 | **P-LOD** (§9) on **S-MINE**; no-drift |
| M3 | L3, L5 | recipe conservation; **S-SMELT** Fine + LOD |
| M4 | L3, L5 | routing conservation, back-pressure, I1; **S-ROUTE** |
| M5 | L3, L5 | chain steady-state, conservation; **S-CHAIN** |
| M6 | L2, L5 | atomic paste, profile equivalence; **S-BLUEPRINT** |
| M7 | L4, L5 | camera isolation `F04-A3`; **S-DETERMINISM** full session |
| M8 | L4, L5 | save/load logical equality; round-trip + transient reconstruction |

(L8 = the determinism harness of §8.)

## 11. Edge cases worth dedicated tests

Derived from the fixed-point and back-pressure rules; easy to get wrong:

- **Rounding carry** (`F01 S03`): fractional rates accumulate and floor with carry — assert
  exact unit counts over long runs, no lost/created units.
- **Deposit depletion to zero** (`F03 S02`): extraction stops exactly at 0, never negative;
  profile output drops to 0.
- **Full-buffer back-pressure** (I2): producers stall, nothing overflows; resume on free
  space.
- **Empty-belt resource rebind** (I1): a belt/road may change `resource` only while empty;
  assert rebind-while-occupied is refused.
- **Starvation scaling** (`F01 S05` AC-3): under-supplied coarse cells scale output down
  proportionally and deterministically.
- **Promotion seeding** (`F01 S06`): first-second output after promotion matches the profile
  within ε — no cold-start dip beyond ε, no created units.

## 12. Out of scope for simulation tests

- Rendering output / pixels, and camera math beyond the isolation property.
- Framework/library internals of the chosen language.
- Any "test" that asserts on internal representation rather than the dump.
- Wall-clock timing/performance as a *correctness* gate (the LOD cost property `F01-A3` is
  reported as a profile, not a pass/fail — `benchmark/scenarios.md` S-PERF).

## 13. Relationship to the benchmark

These tests are the implementor's *internal* mirror of the external rubric in
`benchmark/`. The benchmark axes — A (P-LOD), B (determinism), C (conservation),
D (single-resource), E/F (feature correctness), G (blueprints) — map directly onto §5's
properties and §10's gates. An implementation that passes its own L3–L5 suites should
score well; the benchmark simply re-runs comparable scenarios under controlled
conditions and assigns weights (`benchmark/acceptance.md`).
