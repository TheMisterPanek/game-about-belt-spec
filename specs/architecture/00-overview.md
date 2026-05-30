# Architecture — Overview

This is the big picture. Read it before any feature. It describes *what the system
is made of and how the parts relate*, never how to code it (CONSTITUTION Art. III).

## 1. One simulation, two scopes

```
                         ┌───────────────────────────────────────┐
                         │                WORLD                  │
                         │  (macro grid of Cells + routing)      │
   MapView  ───────────► │                                       │
   (camera, input)       │   ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐     │
                         │   │Cell │ │Cell │ │Road │ │Manu │ ... │
                         │   └──┬──┘ └─────┘ └─────┘ └─────┘     │
                         └──────┼────────────────────────────────┘
                                │  enter / leave
                         ┌──────▼────────────────────────────────┐
   CellView ───────────► │           MICRO GRID (N×N)            │
   (camera, input)       │  miners • belts • assemblers • power  │
                         │  bounded by fixed OUTPUTS              │
                         └───────────────────────────────────────┘
```

- The **macro** layer owns the grid, terrain, occupancy, roads, and macro
  manufacturers.
- The **micro** layer owns one cell's `N×N` factory floor.
- The bridge is the **Production Profile**: a micro layout is reduced to a
  steady-state rate vector that the macro layer consumes. (See `03-simulation-lod.md`.)

## 2. Layered structure (dependency direction points downward)

```
  ┌──────────────────────────────────────────────────────┐
  │  Presentation        rendering, camera, input mapping │  F04
  ├──────────────────────────────────────────────────────┤
  │  Interaction/States  MapView / CellView state machine │  F04
  ├──────────────────────────────────────────────────────┤
  │  Game Systems        building, routing, manufacturing,│  F05–F09
  │                      blueprints                       │
  ├──────────────────────────────────────────────────────┤
  │  Simulation Core     ECS, tick loop, LOD scheduler    │  F01
  ├──────────────────────────────────────────────────────┤
  │  Domain Data         catalog, world/cell/grid model   │  F02, F03
  └──────────────────────────────────────────────────────┘
```

Rule: **upper layers depend on lower ones, never the reverse.** The Simulation Core
knows nothing about cameras or input. The Catalog knows nothing about simulation.
This keeps the spec implementable in any architecture (ECS, actor, plain modules).

## 3. The ECS choice

The simulation is specified as an **Entity-Component-System** model
(`01-ecs-model.md`) because the design needs thousands of cheap, uniform entities
(cells, belt segments, items) updated by a few systems. ECS is described
*abstractly* — as entities (ids), components (plain data), and systems (functions
over component sets). An implementor may realize it with archetypes, sparse sets,
arrays-of-structs, actors, or a relational store, as long as the observable
behavior and the system update order match the spec.

## 4. The performance contract (LOD)

A large world cannot tick every micro entity every frame. The architecture's
central optimization (CONSTITUTION Art. VI) is **two-tier simulation**:

- **Active cells** → fine-grained ECS systems run over their micro entities.
- **Inactive cells** → represented only by a `ProductionProfile`; advanced by
  `rate × Δt`. No per-belt work.

The **LOD Scheduler** (part of F01) decides which cells are active, and performs
**promotion** (profile → live micro entities) and **demotion** (live micro entities
→ profile) losslessly at steady state. This is the hardest correctness requirement
in the project and the most discriminating part of the benchmark.

## 5. Module boundaries

Detailed in `04-module-boundaries.md`. Summary of modules and their one-line jobs:

| Module | Owns | Feature |
|--------|------|---------|
| `catalog` | resource types, tiers, recipes | F02 |
| `world` | macro grid, cells, terrain, generation | F03 |
| `microfactory` | micro grid, buildings, outputs | F05, F06 |
| `routing` | roads / big belts between cells | F07 |
| `macromfg` | macro manufacturers | F08 |
| `blueprint` | capture / paste of layouts | F09 |
| `sim` | ECS world, tick loop, LOD scheduler, profiles | F01 |
| `interaction` | game states, camera, input mapping | F04 |
| `persist` | save / load | F10 (post-MVP) |

## 6. Data flow per tick (informal)

```
input → interaction → mutate world/micro (placement, routing)        [edit-time]
                          │
tick:                     ▼
  sim.LODScheduler.selectActive(camera, pins)
  for each active cell:   run fine-grained micro systems  →  update its profile
  for each inactive cell: integrate profile × Δt
  routing: move resources along roads using cell outputs/inputs
  macromfg: consume routed inputs, produce outputs
  → world state advanced by Δt
render ← world state (read-only)
```

The numbered features formalize each step. Start with **F01** (the simulation
substrate) and **F02/F03** (the data it operates on), then the scope features.

## 7. Validating it

Because the core is headless and deterministic (§2, §4), it is built by **stepping the
simulation and asserting observable state at each tick**. The methodology — test levels,
the step-and-validate loop, property and golden-master/differential testing, and the
P-LOD recipe — is in `05-testing-strategy.md`. The external scoring rubric is `benchmark/`.
