# GLOSSARY

The canonical vocabulary. Every spec uses these terms with **exactly** these
meanings. If you need a new term, define it here first. Terms are grouped, then
alphabetical within a group. `[macro]`, `[micro]`, `[sim]` tag the scope.

## Scopes & Views

- **Macro / Map View** `[macro]` — the top-level scope: the whole `World` rendered
  as a grid of `Cell`s. Where routing and cell-scale building happen.
- **Micro / Cell View** `[micro]` — the interior scope of a single `Cell`: a
  bounded `N×N` factory floor.
- **Scope** — either *macro* or *micro*. A spec statement tagged with a scope
  applies only there unless stated otherwise.

## World & Grid

- **World** `[macro]` — the entire game state: the macro grid, all cells, all
  routing, plus simulation bookkeeping. Created from a `Seed` and a `WorldConfig`.
- **Cell** `[macro]` — one square of the macro grid. Has a `Terrain`, an
  `Occupancy`, and (if developed) an interior `MicroGrid` and a `ProductionProfile`.
- **Coord** — an integer `(x, y)` grid coordinate. Used at both scopes (macro grid
  coords and micro tile coords are both `Coord`, disambiguated by scope).
- **Terrain** `[macro]` — the natural content of a cell. MVP set:
  `Empty`, `Stone`, `Iron`, `Copper`. A resource terrain holds an `Ore Deposit`.
- **Ore Deposit** `[macro]` — the extractable quantity of a resource in a resource
  cell. May be finite or treated as effectively infinite per `WorldConfig`.
- **Occupancy** `[macro]` — what the player has built on a cell: one of
  `Wild` (untouched), `Mine` (a developed micro factory), `Road` (a routing
  segment), `Manufacturer` (a macro-scale assembler). See F03/F07/F08.

## Micro Floor

- **MicroGrid** `[micro]` — the `N×N` interior of a cell. `N` comes from
  `WorldConfig.cellSize` (default 50), fixed for the world's lifetime.
- **Tile** `[micro]` — one square of a `MicroGrid`. Holds at most one `Building`.
- **Building** `[micro]` — a placed machine on the micro floor: `Miner`, `Belt`,
  `Assembler`, `PowerSource`, `PowerPole`, or an `Output` connector. (Extensible
  via the catalog.)
- **Output** `[micro]` — a fixed connector on the micro floor's boundary that
  exports one resource type to the macro `Cell`. A cell has a small, fixed number
  of outputs decided at cell generation; the factory must be *planned around them*.
- **Miner** `[micro]` — a building placed over the cell's ore that emits the cell's
  `Terrain` resource at a rate.
- **Belt** `[micro]` — a directional conveyor tile carrying **one** resource type
  (see CONSTITUTION Art. VII). Moves `Item`s from its input edge to output edge.
- **Assembler** `[micro]` — a building that consumes input resources per a `Recipe`
  and produces an output resource.
- **PowerSource / PowerPole** `[micro]` — generation and distribution of `Power`.
  Buildings that require power are inactive unless powered.
- **Item** `[micro]` — a discrete unit of a resource in transit on the micro floor.

## Resources & Recipes

- **Resource Type** — a named material (`Stone`, `Iron`, `Copper`, `IronPlate`,
  `CopperWire`, …). Defined only in the catalog (F02). Has a `Tier`.
- **Tier** — an integer "level" of a resource. Raw ores are tier 0; each crafting
  step yields a higher tier. Used to talk about "another level of resources".
- **Recipe** — a transformation: an ordered set of input `(Resource, count)`, an
  output `(Resource, count)`, and a `craftTime`. Used by `Assembler` (micro) and
  `Manufacturer` (macro).
- **Catalog** — the single source of truth for all resource types and recipes
  (F02). No other spec hard-codes resource lists.

## Logistics

- **Road** `[macro]` — an `Empty` cell converted into a macro logistics segment.
  Carries **one** resource type, directionally — a "big belt" between cells.
- **Big Belt** `[macro]` — synonym for the logistics behavior of a `Road`: like a
  micro `Belt` but operating cell-to-cell at macro scale and rate.
- **Manufacturer** `[macro]` — a macro-scale `Assembler`: a building placed on an
  `Empty` cell that consumes routed resources and outputs a higher-tier resource,
  enabling cross-cell production chains.
- **Throughput** — a rate, in `units per simulated second`, at which a resource
  flows past a point (a belt, a road, an output).

## Simulation

- **Tick** `[sim]` — one discrete simulation step. Advances logical state by a
  `Δt` (delta-time).
- **Δt / Delta-Time** `[sim]` — the simulated time advanced in one tick. May be
  fixed or variable per the simulation spec; simulation-affecting state is integer
  / fixed-point (CONSTITUTION Art. IV).
- **Fine-grained** `[sim]` — per-entity simulation (each belt, item, machine ticks
  individually). Used for *active* cells.
- **Coarse-grained** `[sim]` — profile-based simulation: a cell advances by
  `ProductionProfile.rate × Δt` without simulating its interior. Used for
  *inactive* cells. (CONSTITUTION Art. VI.)
- **Active Cell** `[sim]` — a cell currently simulated fine-grained (the player is
  inside it, or it is pinned active). All others are inactive.
- **Production Profile** `[macro/sim]` — the steady-state output (and input demand)
  rate vector a cell's micro layout produces, computed from that layout. The bridge
  between micro and macro (CONSTITUTION Art. V & VI).
- **Promotion / Demotion** `[sim]` — moving a cell between fine- and coarse-grained.
  Must be lossless at steady state within tolerance.

## Interaction

- **Game State** — the top-level mode driving input and rendering: `MapView` or
  `CellView` (plus transitional states). See F04.
- **Camera** — the viewport over the current scope. Moves on `WASD`; the mouse
  selects/places. Pure rendering concern; never affects simulation.
- **Blueprint** — a captured, position-independent copy of a `Cell`'s contents
  (micro layout and/or macro role) that can be pasted onto another cell (F09).
