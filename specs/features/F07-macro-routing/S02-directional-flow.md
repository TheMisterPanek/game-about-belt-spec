# F07 · S02 — Single-Resource Directional Flow

**Feature:** F07 — Macro Routing: Roads & Big Belts
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Move one resource type along a road from a source cell's output to a downstream
consumer — the "big belt" behavior, macro analogue of micro belts.

## Behavior
1. `RoutingSystem` (schedule step 2) moves the road's bound `resource` from the
   upstream source (`MacroOutput.buffer` of the source cell, or an upstream road) into
   the road's `buffer`, then onward to the downstream cell's intake (a `Manufacturer`
   input, another road, or a consuming cell's demand) following `dir`.
2. A road carries exactly one resource type (Invariant I1); it never interleaves two.
3. Movement is `Fixed`-rate, integer-unit, conserving material — units enter, traverse,
   and exit without loss.
4. Routing runs every tick for all roads regardless of LOD tier (roads are always
   coarse and cheap).

## Data touched
`RoadSegment.buffer`, source/destination `MacroOutput.buffer` and manufacturer inputs.

## Acceptance criteria
- **AC-1** *Given* a source with the road's resource buffered, *then* units flow along
  `dir` into the downstream intake over time.
- **AC-2** *Then* the road never holds a second resource type (Invariant I1).
- **AC-3** *Over* a run, units entering the road = units delivered + units still in its
  buffer (conservation).

## Out of scope
Throughput caps/back-pressure (S03); multi-road splitting (S04).
