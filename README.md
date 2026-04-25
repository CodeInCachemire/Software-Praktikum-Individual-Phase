# 🌊 Ocean Cleanup Simulation — Individual Phase
**Saarland University · Softwarepraktikum (SoPra) · Kotlin**

> Extending an unfamiliar production-style simulation codebase independently,
> under strict architectural, testing, and code-quality requirements.

[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.24-7F52FF?style=flat&logo=kotlin)](https://kotlinlang.org)
[![JDK](https://img.shields.io/badge/JDK-17-orange?style=flat&logo=openjdk)](https://openjdk.org)
[![Gradle](https://img.shields.io/badge/Gradle-Build-02303A?style=flat&logo=gradle)](https://gradle.org)
[![JUnit](https://img.shields.io/badge/JUnit-5-25A162?style=flat&logo=junit5)](https://junit.org)
[![Detekt](https://img.shields.io/badge/Detekt-Static_Analysis-blueviolet?style=flat)](https://detekt.dev)

---

## Result

✅ **48/50 server tests passed · 8/10 mutants detected**
Achieved despite approximately 25% of the official specification containing
contradictions, omissions, or ambiguous edge cases — all of which required
independent interpretation and documented design decisions. 
Due to this the following iteration of the course got rid of this phase entirely. 

---

## Overview

This is my individual-phase submission for the SoPra Software Engineering Lab
at Saarland University. The task was not to build from scratch, but to:

1. Analyzed an unfamiliar codebase related to the group project topic
2. Understand its architecture
3. Extend it with substantial new functionality across multiple layers
4. Deliver a formal UML class diagram and written project description
   documenting every design decision
5. Write unit, integration, and system tests sufficient to detect
   injected faults in a faulty reference server implementation

Group work was strictly prohibited. Every design decision, line of code,
and test was my own.

---

## Design & Documentation

This project was submitted with:

- **UML Class Diagram** covering all new and modified classes,
  their relationships, and integration points with unchanged components
- **Written Project Description** documenting the rationale behind
  every design decision, pattern usage, and deviation from the
  initial design after implementation

### Key Design Decisions

**Damaged ship behavior as a state machine**
Typhoon impact triggers a state transition affecting velocity and
acceleration properties. The ship's decision logic changes priority order
(damaged → seek shipyard, above all except refuel and restricted-area exit).
On repair completion, properties are restored. This was modeled explicitly
rather than through scattered conditionals, making the behavior traceable
and testable in isolation.

**Credit deduction ordering**
The spec requires a precise within-tick ordering: refuel → repair → purchase.
Each check evaluates affordability at decision time against current balance,
not projected balance. Separating the "decide" and "execute" phases within
each corporation's tick loop was essential to get this right.

**Refueling ship autonomy**
Before moving toward a low-fuel ship, a refueling ship must verify it has
enough fuel to return to a refueling station from the *destination tile*,
not its current position. This lookahead check required clean access to
pathfinding results without duplicating routing logic.

**Parser extension without regression**
New JSON structures were added in a way that kept existing valid
configurations valid. Cross-file validation (e.g., harbor IDs referenced
in corporations must exist in the map) was implemented as a separate
validation pass after all three files were parsed.

---

## Engineering Challenges

**Incomplete specification**
Around a quarter of the given specification was contradictory or incorrectly defined,
  which caused the benchmark server side test suite to be incorrect, requiring careful engineering that satisfied the test suite
  without violating the correct behavior elsewhere.
  
**Deterministic correctness**
Server-side tests compare exact log output. A single out-of-order log line
or off-by-one in credit arithmetic fails the test. This required careful
attention to execution order throughout the entire tick loop.

**Behavioral surface area**
A single new feature (e.g., refueling ships) touched movement logic,
fuel calculations, pathfinding, logging, parsing, validation, and tests.
Keeping changes isolated while ensuring correct interaction required
disciplined use of existing module boundaries.

---
## What I Extended

The base simulation models corporations cleaning garbage from a hexagonal
ocean grid using ships. My extension added:

| Feature | Description |
|---|---|
| **Credit Economy** | Corporations earn credits by unloading garbage; spend them on repairs, refuels, and ship purchases. All actions are credit-gated with precise per-tick deduction ordering |
| **Harbors as Entities** | Harbors migrated from boolean tile flags to first-class entities with configurable stations, ownership, and location |
| **Shipyard Station** | Ships can be repaired here (1 tick, credit cost). Corporations can purchase new refueling ships with configurable delivery delays |
| **Refueling Station** | Ships refuel fuel tanks or refueling capacity here, with limited usage counts before the station deactivates |
| **Unloading Station** | Corporation-exclusive; ships unload collected garbage and earn credits per kg/liter |
| **Refueling Ships** | Autonomous ships purchased mid-simulation. They track nearby low-fuel ships, manage their own fuel reserves, and decide whether they can safely reach a refueling station before committing to a movement |
| **Typhoon Events** | Four-tier cascading effect system: fuel consumption increase → instrument destruction → garbage unloading → ship damage with modified kinematics until repaired |
| **Damaged Ship FSM** | Typhoon-damaged ships enter a degraded state (reduced max velocity and acceleration), reroute to the nearest shipyard, and restore properties on successful repair |
| **Extended JSON Parsing** | Modified parser and validator to handle new harbor entities, station properties, corporation credits, typhoon events, and refueling ship configuration — with both intra-file and cross-file consistency checks |
| **Updated Ship Behavior** | Existing ship types gained new decision branches: respond to nearby refueling ships, reroute on damage, handle credit-insufficient scenarios gracefully |

---

## Architecture
```
src/
├── model/        # Entities: ships, tiles, harbors, stations, garbage, events
├── controller/   # Tick loop, corporation logic, ship FSM, event dispatch
├── parser/       # JSON parsing + intra and cross-file schema validation
├── logger/       # Structured deterministic log output
└── test/         # Unit, integration, and system tests
```

The tick execution order is strictly deterministic:
```
For each corporation (ascending ID):
  1. Move ships
  2. Attach trackers
  3. Collect garbage
  4. Cooperate with other corporations
  5. Refuel ships          ← credits deducted first
  6. Repair damaged ships  ← credits deducted second
  7. Purchase refueling ships ← credits deducted last

Then (globally):
  8. Garbage drift
  9. Ship drift
  10. Events occur
  11. Tasks updated
```

---

## Testing

| Type | Focus |
|---|---|
| **Unit** | Ship FSM transitions, credit arithmetic, station capacity logic, damage property changes |
| **Integration** | Multi-component scenarios: typhoon → damage → pathfinding → shipyard → repair → restored behavior |
| **System** | Full simulation runs validated against exact expected log output |
| **Mutation** | Targeted system tests designed to catch behavioral faults in a faulty reference implementation — 8/10 injected faults detected |

---

## Running the Simulation
```bash
./gradlew build

java -jar build/libs/simulation.jar \
  --map path/to/map.json \
  --corporations path/to/corporations.json \
  --scenario path/to/scenario.json \
  --max_ticks 500 \
  --out output.log
```

---

## Keywords
Kotlin · Gradle · JUnit · Detekt · Object-Oriented Design ·
Finite State Machines · Event-Driven Simulation · JSON Parsing ·
Cross-Module Integration · Deterministic Systems · Mutation Testing ·
University Project · Saarland University
