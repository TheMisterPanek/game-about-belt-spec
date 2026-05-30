# F08 · S03 — Integration with Routing (Demand & Supply)

**Feature:** F08 — Macro Manufacturing
**Scope:** macro · **MVP:** yes · **Depends on:** S02, F07 S02

## Intent
Wire a manufacturer into the road network as both a demand sink (for its inputs) and a
supply source (for its output), so it participates correctly in flow and back-pressure.

## Behavior
1. A manufacturer presents an input demand per its recipe; inbound roads carrying those
   resources deliver into its `inputs` buffers, subject to road throughput (F07 S03).
2. Its `output` feeds the cell's `MacroOutput.buffer`, picked up by outbound roads. When
   the output buffer is full, the manufacturer stalls (back-pressure, I2).
3. Demand and supply are deterministic; the manufacturer neither creates nor loses
   material at the road boundary.

## Data touched
`MacroManufacturer.inputs/output`, inbound/outbound `RoadSegment`, `MacroOutput.buffer`.

## Acceptance criteria
- **AC-1** *Given* inbound roads of its input resources, *then* the manufacturer's
  `inputs` fill at the delivered rate (bounded by road throughput).
- **AC-2** *Given* a full output buffer with no outbound road, *then* the manufacturer
  stalls; adding an outbound road resumes it.
- **AC-3** *Then* material crossing the road↔manufacturer boundary is conserved.

## Out of scope
Recipe math (S02); cross-cell chains (S04); routing internals (F07).
