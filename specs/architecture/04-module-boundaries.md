# Architecture — Module Boundaries

Modules are conceptual; an implementation may package them as crates, packages,
namespaces, or folders. What is binding is **who depends on whom** and **what each
exposes** — so the system stays implementable in any architecture and any language.

## 1. Dependency graph (arrows = "depends on")

```
  interaction ──► sim ──► world ──► catalog
       │           │       ▲          ▲
       │           │       │          │
       └──► (renders, never mutates sim except via commands)
   microfactory ──► world, catalog
   routing      ──► world, catalog
   macromfg     ──► world, catalog
   blueprint    ──► microfactory, world, catalog
   sim          ──► microfactory, routing, macromfg (drives their systems)
   persist      ──► world, catalog (+ all state owners)
```

Hard rules:
- `catalog` depends on **nothing** (pure data). It is the root.
- `world` depends only on `catalog`.
- No lower module may reference `interaction` (presentation must be removable).
- Game-logic modules (`microfactory`, `routing`, `macromfg`) never call each other
  directly; they coordinate **through `world` state** and are sequenced by `sim`.

## 2. Module responsibilities & public surface

Each module exposes **commands** (state mutations, applied between ticks),
**systems** (tick-time behavior, owned by `sim`'s schedule), and **queries**
(read-only views). Listed as contracts, not signatures.

### catalog (F02)
- Queries: `resource(id)`, `recipe(id)`, `allResources()`, `resourcesByTier(t)`,
  `recipesProducing(resource)`.
- No commands at runtime (catalog is authored data, loaded once, immutable).

### world (F03)
- Commands: `generate(WorldConfig) → World`, `setOccupancy(coord, Occupancy)`,
  `developCell(coord) → MicroGrid`.
- Queries: `cellAt(coord)`, `neighbors(coord)`, `outputsOf(coord)`.

### microfactory (F05, F06)
- Commands: `place(cell, tile, BuildingKind, dir)`, `remove(cell, tile)`,
  `configureAssembler(cell, tile, RecipeId)`.
- Systems: `PowerSystem`, `MinerSystem`, `BeltSystem`, `AssemblerSystem`,
  `OutputSystem`, `ProfileUpdateSystem`.
- Queries: `gridOf(cell)`, `tileAt(cell, coord)`, `profileOf(cell)`.

### routing (F07)
- Commands: `layRoad(coord, dir, resource)`, `clearRoad(coord)`.
- Systems: `RoutingSystem`.
- Queries: `roadAt(coord)`, `flowInto(coord)`.

### macromfg (F08)
- Commands: `placeManufacturer(coord, RecipeId)`, `removeManufacturer(coord)`.
- Systems: `MacroManufactureSystem`.
- Queries: `manufacturerAt(coord)`.

### blueprint (F09)
- Commands: `capture(cell) → Blueprint`, `paste(Blueprint, cell)`.
- Queries: `canPaste(Blueprint, cell) → Result`.

### sim (F01)
- Owns the ECS world, the tick loop, the schedule, and the LOD scheduler.
- Commands: `tick(Δt)`, `pinActive(coord)`, `unpinActive(coord)`.
- Systems (its own): `LODSchedulerSystem`, `CoarseProductionSystem`.
- Queries: `activeSet()`, `tickNo()`.

### interaction (F04)
- Owns game states (`MapView`/`CellView`), camera, and input→command mapping.
- Calls into other modules **only** via their commands/queries; never reaches into
  their internal state.
- Produces no simulation state — purely translates intent into commands and reads
  state to render.

### persist (F10, Post-MVP)
- Commands: `save(World) → Bytes`, `load(Bytes) → World`.
- Must round-trip every other module's persistent state; transient micro entities
  are reconstructed via promotion, not serialized (see `03-simulation-lod.md`).

## 3. Command vs system discipline

- **Commands** mutate state **between** ticks (player actions, blueprint paste). They
  validate against invariants and either fully apply or fully reject.
- **Systems** mutate state **during** a tick, in schedule order, and never perform
  player-level validation (that already happened at command time).
- This split is what makes the simulation deterministic and replayable: a session is
  `generate(config)` + an ordered stream of `(tickNo, command)` events.

## 4. Testability seam

Because `interaction` and `persist` sit at the edges and everything routes through
`world` + the `sim` schedule, a **headless** configuration (no camera, no rendering)
must be able to: generate a world, apply a command script, run N ticks, and dump
logical state. The benchmark relies on this seam (`benchmark/README.md`).
