# Architecture — Data Model (neutral pseudo-notation)

All shared data structures live here, in a **language-neutral pseudo-notation**.
Implementations map these to their own types. The *names*, *fields*, *units*, and
*invariants* are binding; the representation is free (CONSTITUTION Art. III).

## 0. Notation

```
Type Name {
  field: Kind            # comment / unit / invariant
}
enum Name { A, B, C }
alias Name = Kind
Vec<T>      ordered list of T
Map<K,V>    association from K to V
Opt<T>      T or absent
Id          opaque unique identity (entity id)
Int         exact integer
Fixed       fixed-point rational, scale defined in 03-simulation-lod.md
Real        real number — PRESENTATION ONLY, never simulation state
Coord       { x: Int, y: Int }
ResourceId  Int        # index into the catalog
RecipeId    Int        # index into the catalog
```

> **Units convention.** Rates are `units per simulated second` as `Fixed`. Amounts
> and counts are `Int`. Time is `Fixed` seconds. Never store a rate or amount as
> `Real` — that is a determinism violation.

## 1. Configuration & world

```
Type WorldConfig {
  seed:        Int
  mapWidth:    Int          # cells, default 256
  mapHeight:   Int          # cells, default 256
  cellSize:    Int          # N for the N×N micro grid, default 50, fixed for world life
  oreFinite:   Bool         # true → deposits deplete; false → effectively infinite
}

Type World {
  config:  WorldConfig
  cells:   Map<Coord, Id>   # macro grid: coord → cell entity
  tickNo:  Int              # monotonically increasing tick counter
}
```

## 2. Macro cell

```
enum Terrain { Empty, Stone, Iron, Copper }      # MVP set; extended via catalog tags
enum Occupancy { Wild, Mine, Road, Manufacturer }

Type OreDeposit {
  resource:  ResourceId
  remaining: Int            # ignored when WorldConfig.oreFinite == false
}

Type Cell {                 # realized as CellTag + components on one entity
  coord:     Coord
  terrain:   Terrain
  deposit:   Opt<OreDeposit>     # present iff terrain is a resource
  occupancy: Occupancy
  micro:     Opt<Id>             # micro grid root entity, present iff developed
  profile:   Opt<ProductionProfile>
  outputs:   Vec<MacroOutput>    # fixed at generation; what this cell can export
}

Type MacroOutput {
  resource:  ResourceId
  buffer:    Int            # units waiting to be picked up by a Road
  capacity:  Int            # max buffer; production stalls when full
}
```

## 3. Production profile (micro ↔ macro bridge)

```
Type RateEntry { resource: ResourceId, rate: Fixed }   # units / simulated second

Type ProductionProfile {
  outputs: Vec<RateEntry>   # steady-state production this cell exports
  demands: Vec<RateEntry>   # steady-state inputs this cell needs imported
  valid:   Bool             # false → must be recomputed before coarse use
}
```

A profile is the *reduced* behavior of a micro layout. For a pure mining cell with
one miner of rate `r` feeding one output, `outputs = [{ore, r}]`, `demands = []`.
For a cell that imports `Iron` and exports `IronPlate`, both vectors are non-empty.

## 4. Micro grid & buildings

```
Type MicroGrid {
  cell:    Id               # owning macro cell
  size:    Int              # == WorldConfig.cellSize
  tiles:   Map<Coord, Id>   # micro coord → building entity (absent = empty tile)
  outputs: Vec<Id>          # the OutputPort entities on the boundary
}

enum BuildingKind { Miner, Belt, Assembler, PowerSource, PowerPole, Output }
enum Direction { North, East, South, West }

Type BeltState {
  dir:      Direction
  resource: Opt<ResourceId>   # the single type carried (Art. VII); absent = empty belt
  slots:    Vec<Opt<ResourceId>>  # fixed-length item track; positions along the belt
}

Type MinerState {
  resource: ResourceId        # equals owning cell's ore
  rate:     Fixed             # units / second at full power
  buffer:   Int
}

Type AssemblerState {
  recipe:   RecipeId
  inputs:   Map<ResourceId, Int>   # accumulated input units
  output:   Int                    # finished output units waiting to leave
  progress: Fixed                  # 0..craftTime
}

Type PowerState {
  produced:  Int             # by PowerSource
  consumed:  Int             # by powered consumers
  available: Int             # network-wide this tick
  powered:   Bool            # for a consumer: is it receiving power this tick
}

Type OutputPort {
  pos:      Coord            # boundary tile
  resource: ResourceId       # what this port exports
  macroOut: Int             # index into Cell.outputs it feeds
}
```

## 5. Macro logistics & manufacturing

```
Type RoadSegment {
  resource: Opt<ResourceId>  # single type carried (Art. VII)
  dir:      Direction
  buffer:   Int              # units in transit on this segment
  rate:     Fixed            # max throughput, units / second
}

Type MacroManufacturer {
  recipe:   RecipeId
  inputs:   Map<ResourceId, Int>
  output:   Int
  progress: Fixed
}
```

## 6. Catalog (see F02 for authoring rules)

```
Type ResourceDef {
  id:    ResourceId
  name:  String
  tier:  Int                 # 0 = raw ore
}

Type RecipeDef {
  id:        RecipeId
  inputs:    Vec<{ resource: ResourceId, count: Int }>
  output:    { resource: ResourceId, count: Int }
  craftTime: Fixed           # simulated seconds
}

Type Catalog {
  resources: Vec<ResourceDef>
  recipes:   Vec<RecipeDef>
}
```

## 7. Blueprint (see F09)

```
Type Blueprint {
  size:      Int                    # source cellSize; paste requires match
  buildings: Vec<{ rel: Coord, kind: BuildingKind, data: BuildingData }>  # position-independent
  role:      Opt<MacroRole>         # captured macro occupancy (Road/Manufacturer) if any
}
```

(`BuildingData` is the variant payload — one of `BeltState`/`MinerState`/… with
absolute positions stripped. `MacroRole` mirrors `Occupancy` for macro pastes.)

## 8. Invariants (cross-cutting)

- **I1** A `Belt` / `RoadSegment` whose `resource` is set never carries a different
  resource until it empties (`resource` may only change while `buffer`/`slots` are
  empty). (CONSTITUTION Art. VII)
- **I2** `0 ≤ MacroOutput.buffer ≤ capacity`. Production that would exceed capacity
  is *back-pressured* (stalls), never discarded.
- **I3** A cell has `micro != absent` **iff** `occupancy == Mine`. Roads and
  Manufacturers have no micro grid.
- **I4** Micro entities exist only while the owning cell has `ActiveTag`. When
  absent, the cell is represented by `profile` (which must have `valid == true`).
- **I5** Every `OutputPort.resource` and every `MacroOutput.resource` exists in the
  Catalog; every `RecipeId`/`ResourceId` is a valid catalog index.
- **I6** Fixed-point arithmetic uses the single global scale from
  `03-simulation-lod.md`; mixing scales is forbidden.
