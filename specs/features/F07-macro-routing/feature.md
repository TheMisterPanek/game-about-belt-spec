# F07 — Macro Routing: Roads & Big Belts

**Module:** `routing` · **Scope:** macro · **MVP:** yes

## Intent

Move resources *between* cells. An `Empty` cell can be converted into a `Road`: a
macro-scale conveyor — a **Big Belt** — that carries **one** resource type
directionally from a source cell's output toward a downstream consumer (another
cell's demand, a macro `Manufacturer`, or a chain of further roads). This is the
macro analogue of micro belts and obeys the same single-resource rule
(CONSTITUTION Art. VII).

## Scope

In: laying/clearing roads on empty cells, road direction & single-resource binding,
the `RoutingSystem` that advances flow between `MacroOutput` buffers and downstream
inputs, throughput limits and back-pressure across a road network. Out: micro belts
(F06), what consumes the resource (F08), and what produces it (F05).

## Data

`RoadSegment`, `MacroOutput` (`02-data-model.md` §2,§5). System: `RoutingSystem`
(`01-ecs-model.md` §4, schedule step 2).

## Rules of note

- A road carries one resource type (I1); to carry a different one it must first be
  cleared/empty.
- Roads connect orthogonally adjacent cells; flow direction is explicit (`dir`).
- Throughput is capped by `RoadSegment.rate`; excess upstream is back-pressured into
  the source `MacroOutput.buffer` (which itself stalls production at capacity, I2).
- Routing is a macro system: it runs every tick over **all** roads regardless of LOD
  tier — roads are cheap and always coarse.
- Flow is deterministic: a junction with multiple downstream demands splits by a
  documented rule (e.g. round-robin by ascending neighbor coord) so results match
  across implementations.

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Lay & clear roads on empty cells | ✅ |
| S02 | Single-resource directional flow | ✅ |
| S03 | Throughput limits & back-pressure | ✅ |
| S04 | Multi-cell routing networks & deterministic splitting | ✅ |

## Feature-level acceptance

- **F07-A1** A road may be laid only on an `Empty`, `Wild` cell; laying sets its
  occupancy to `Road`; clearing returns it to `Empty/Wild`.
- **F07-A2** A road never carries two resource types at once; rebinding requires it to
  be empty (Invariant I1).
- **F07-A3** Over a long run, units entering a road network equal units delivered plus
  units still buffered — conservation holds end-to-end.
- **F07-A4** A fixed network with fixed inputs delivers the same per-cell totals every
  run (determinism), including split decisions at junctions.
