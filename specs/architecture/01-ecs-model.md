# Architecture — ECS Model (abstract)

The simulation is specified as an Entity-Component-System. This document defines the
ECS **abstractly**, as a contract on behavior and update order — not as a particular
data structure or library (CONSTITUTION Art. III).

## 1. Definitions

- **Entity** — an opaque, unique identity (an id). Carries no data itself.
- **Component** — a named bag of plain data attachable to an entity. A given
  component type is attached at most once per entity. Components contain *no
  behavior*.
- **System** — a pure-ish function `System(world, Δt)` that reads and writes
  components of entities matching a declared **query** (a set of required/excluded
  component types). Systems are the only place simulation logic lives.
- **ECS World** — the container of all entities and components plus the ordered
  system schedule. (Distinct from the game `World` of F03, which is *domain* data
  stored *as* entities/components. When ambiguous, say "ECS world" vs "game World".)

An implementor may store components however they like (arrays, maps, archetypes).
The spec constrains only: which components exist, which systems read/write them, and
**the order systems run in**.

## 2. Determinism rules

1. Systems run in a **fixed, declared order** each tick (the *schedule*, §5).
2. Within a system, entities are processed in a **deterministic order**: ascending
   entity id, unless the system spec states another key. No iteration over
   hash-unordered collections in a way that affects results.
3. Simulation-affecting component fields are **integer / fixed-point**
   (CONSTITUTION Art. IV). Real-valued fields may exist only for
   presentation-only components (e.g. interpolated render position) and must never
   feed back into simulation.

## 3. Core component families

Components are grouped by concern. Exact field lists live in `02-data-model.md`;
here is the catalog of component *types* and which feature owns them.

### Macro (game World) components
- `CellTag` — marks an entity as a macro cell; holds its `Coord`. (F03)
- `Terrain` — the cell's natural content + ore deposit. (F03)
- `Occupancy` — Wild / Mine / Road / Manufacturer. (F03)
- `ProductionProfile` — steady-state input demand + output rate vectors. (F01/F05)
- `RoadSegment` — resource type carried, direction, buffered amount, rate. (F07)
- `MacroManufacturer` — recipe id, input buffers, output buffer, progress. (F08)
- `ActiveTag` — present iff the cell is currently fine-grained. (F01)

### Micro (factory floor) components
- `MicroOf` — back-reference to the owning macro cell entity. (F05)
- `TilePos` — `Coord` on the micro grid. (F05)
- `BuildingKind` — Miner / Belt / Assembler / PowerSource / PowerPole / Output. (F05/F06)
- `BeltState` — direction, carried resource type, item slots. (F06)
- `MinerState` — output resource, rate, internal buffer. (F06)
- `AssemblerState` — recipe id, input buffers, output buffer, progress. (F06)
- `PowerState` — produced/consumed/available power, powered flag. (F06)
- `OutputPort` — boundary connector: resource type, link to macro cell output. (F05)

### Cross-cutting
- `Dirty` — marks an active cell whose profile must be recomputed. (F01)

> Micro entities exist **only for active cells**. Demotion deletes them; promotion
> re-creates them from the layout. See `03-simulation-lod.md`.

## 4. Systems (per scope)

Each system has: a query, inputs (component reads), outputs (component writes), and
a one-line contract. Full behavior is in the owning feature. Listed here in
schedule order.

### Macro / coordination systems (run every tick)
1. `LODSchedulerSystem` — decide active set from camera + pins; promote/demote;
   add/remove `ActiveTag`. (F01)
2. `RoutingSystem` — move resources along `RoadSegment`s between cell outputs and
   downstream inputs, respecting single-resource and throughput rules. (F07)
3. `MacroManufactureSystem` — advance `MacroManufacturer`s by recipe × Δt. (F08)
4. `CoarseProductionSystem` — for inactive cells, integrate
   `ProductionProfile.rate × Δt` into their output buffers. (F01)

### Micro / fine-grained systems (run only over `ActiveTag` cells' micro entities)
5. `PowerSystem` — compute available power; set `powered` on consumers. (F06)
6. `MinerSystem` — powered miners emit resource into adjacent belt/buffer. (F06)
7. `BeltSystem` — advance items along belts; hand off at junctions/outputs. (F06)
8. `AssemblerSystem` — consume inputs, advance progress, emit outputs. (F06)
9. `OutputSystem` — push items reaching `OutputPort`s into the macro cell output. (F05)
10. `ProfileUpdateSystem` — for `Dirty` active cells, recompute `ProductionProfile`
    from observed steady-state so demotion is lossless. (F01/F05)

## 5. The schedule

The canonical per-tick order is **1 → 10** as numbered in §4. Rationale:

- LOD first, so the active set is settled before anyone reads it.
- Macro routing/manufacture and coarse production advance the world skeleton.
- Fine-grained micro systems run in *physical* order: power gates everything;
  miners create items; belts move them; assemblers transform; outputs export;
  profile recompute closes the loop.

Implementations may parallelize *within* a system if and only if the result is
identical to the deterministic serial order (§2).

## 6. What ECS does **not** dictate

- Rendering, input, and camera are **not** systems in this schedule; they live in
  the interaction layer (F04) and run outside the deterministic tick.
- Edit-time mutations (placing a building, drawing a road) are applied between
  ticks via documented commands, not by simulation systems.
