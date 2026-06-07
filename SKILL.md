---
name: sysml-v2-modeling
description: Author and review Model-Based Systems Engineering models in OMG SysML v2 (built on KerML), textual notation first. Covers the definition-usage pattern, structure, behavior, requirements-as-constraints, allocation, the standard library, views, and language extension via metadata. Use whenever creating, editing, migrating, or reviewing SysML v2 (.sysml) or KerML (.kerml) models.
license: CC-BY-4.0
version: 1.1.0
allowed-tools: Read, Glob, Grep, Write, Edit
---

# SysML v2 Modeling Skill

A concise, specification-grounded guide for working in **SysML v2** - the OMG Systems Modeling Language built on **KerML** - with the **textual notation as the primary, normative form**.

## Specification Baseline

This skill targets **OMG SysML v2.0** (and its KerML foundation), with notation following the SysML v2 release train as of **2026-01**. SysML v2 notation still evolves between releases; when a construct is unfamiliar or release-sensitive, **verify against the current OMG specification and monthly release** rather than relying on memory.

## Installing & Using This Skill

- **As an agent skill:** place this file as `SKILL.md` in a folder named `sysml-v2-modeling/` inside your agent's skills directory. The agent should load it whenever SysML v2 / KerML work is detected.
- **As context:** paste the contents into your assistant's system or context window before SysML v2 tasks.
- **Pair with the spec:** for deep or edge-case syntax, consult the OMG SysML v2 / KerML specifications and the OMG "Introduction to the SysML v2 Language" documents (see *References*).

## When to Use This Skill

- **SysML v2 modeling** - authoring or editing `.sysml` models (or `.kerml`).
- **MBSE work** - requirements, structure, behavior, constraints, analysis, and allocation for a system of any kind (hardware, software, human, process).
- **Migrating from SysML v1** - re-expressing v1 Blocks/BDD/IBD/profile content as v2 definitions and usages.
- **Best practices** - following the v2 paradigm rather than carrying v1/UML habits forward.

> This skill is for **SysML v2**, not v1. They are different languages with different metamodels - not two notations for the same thing. If you are reaching for Block stereotypes, BDD vs IBD, or PlantUML drawing syntax, that is v1 and the wrong mental model here.

## Overview

SysML v2 differs from v1 in four foundational ways:

1. **Textual notation is normative and primary.** The model *is* the text. Diagrams are *views* rendered from the same model, not the source of truth.
2. **One model, one semantic graph.** There is no BDD/IBD split. A part, its internals, and its connections are the same elements seen through different views.
3. **No stereotypes / no UML profiles.** Every concept is a first-class keyword (`part`, `port`, `action`, `requirement`, …). Extension is done with **model libraries + metadata**, not by stereotyping a UML base.
4. **Formal semantics.** Requirements are evaluable constraints; structure, behavior, and quantities have defined semantics, enabling analysis, verification, and a standardized API.

## CRITICAL: The Definition-Usage Pattern

The single most important idea in SysML v2. Almost every element comes in two forms:

- A **definition** (`<kind> def Name`) - the reusable *type/classifier* (like a class).
- A **usage** (`<kind> name : Definition`) - an *occurrence/role* of a definition in a context (like an instance, role, or property).

```sysml
part def Engine;                     // definition: the kind "Engine"
part def Vehicle {
    part engine : Engine;            // usage: this Vehicle has one engine
    part wheels[4] : Wheel;          // usage with multiplicity
    ref part driver[0..1] : Person;  // referential (non-composite) usage
}
```

The pattern is uniform across structure **and** behavior: `attribute`/`attribute def`, `item`/`item def`, `port`/`port def`, `action`/`action def`, `state`/`state def`, `constraint`/`constraint def`, `requirement`/`requirement def`, `connection`/`connection def`, `interface`/`interface def`, `view`/`view def`. Learn it once; it applies everywhere.

Multiplicity defaults to `[1..1]` for attribute, item, and part usages. Composite usages are owned; add `ref` for referential (shared, non-owning) usages.

## SysML v2 vs SysML v1

| Aspect | SysML v1 | SysML v2 |
|--------|----------|----------|
| Source of truth | Diagrams (graphical) | Textual model (graphical is a view) |
| Foundation | UML profile | KerML metamodel + formal semantics |
| Core building block | Block | Definition / Usage |
| Diagram kinds | BDD, IBD (separate) | One model; interconnection is a view |
| Extension | UML stereotypes / profiles | Model libraries + metadata + user-defined keywords |
| Requirements | Text boxes with attributes | Evaluable constraints (subject, assume, require, satisfy, verify) |
| Quantities/units | Value types | Standard ISQ/SI library with quantity values and unit literals |
| Interoperability | XMI (tool-variable) | Standardized Systems Modeling API + JSON interchange |

### Terminology mapping (v1 → v2)

| SysML v2 | SysML v1 |
|----------|----------|
| `part` / `part def` | part property / block |
| `attribute` / `attribute def` | value property / value type |
| `port` / `port def` | proxy port / interface block |
| `action` / `action def` | action / activity |
| `state` / `state def` | state / state machine |
| `constraint` / `constraint def` | constraint property / constraint block |
| `requirement` / `requirement def` | requirement |
| `connection` / `connection def` | connector / association block |
| `view` / `view def` | view |

## Textual Notation Primer

### Packages, imports, visibility

```sysml
package VehicleModel {
    private import ScalarValues::*;     // pulled in, not re-exported
    public  import ISQ::*;              // re-exported to importers of this package

    part def Vehicle { /* ... */ }
}
```

Import forms: `Pkg::Name` (one member), `Pkg::*` (members), `Pkg::**` (recursive, into nested namespaces). Visibility keywords: `public`, `private`, `protected`.

### Documentation and comments

```sysml
part def Engine {
    doc /* The vehicle's prime mover. */
    // line comment
    /* block comment */
}
```

### Specialization, redefinition, subsetting

| Operator | Keyword | Meaning |
|----------|---------|---------|
| `:>` | `subsets` | usage is a subset of another usage / specializes a definition |
| `:>>` | `redefines` | usage overrides an inherited feature |
| (typing `:`) | `specializes` | definition specializes a more general definition |

```sysml
part def Axle { attribute mass : ISQ::MassValue; }
part def FrontAxle :> Axle {            // specializes Axle (inherits mass)
    attribute steeringAngle : ISQ::AngleValue;
}

part def Engine { part cylinders[4..8] : Cylinder; }
part def Engine4Cyl :> Engine {
    part cylinders[4] :>> cylinders;    // redefine multiplicity to exactly 4
}
```

## Structure

### Attributes, items, parts

```sysml
attribute def Temperature :> ISQ::ThermodynamicTemperatureValue;

item def Fuel {
    attribute pressure : ISQ::PressureValue;
    ref item impurities[0..*] : Material;
}

part def Vehicle {
    attribute mass : ISQ::MassValue;
    part engine : Engine;
    part wheels[4] : Wheel;
    ref part driver[0..1] : Person;     // referenced, not owned
}
```

- **attribute** - data-valued feature (typed by an `attribute def` or library quantity).
- **item** - something acted on, stored, or flowing (matter, energy, data); can be continuous.
- **part** - a modular unit of structure (a kind of item). A part can be treated as an item when it flows (e.g., an engine moving down an assembly line).

### Quantities and unit literals

```sysml
attribute mass : ISQ::MassValue = 1500 [kg];
attribute thrust : ISQ::ForceValue = 5000 [N];
attribute wheelDiameter : ISQ::LengthValue = 33 ['in'];
```

Unit literals use square brackets: `1500 [kg]`, `30 [mi / gallon]`, `48 [h]`. Quantity types live in the **ISQ**/**SI**/**Quantities** standard libraries.

## Ports, Interfaces, Connections, Flows

### Ports (replace v1 proxy ports / interface blocks)

```sysml
port def FuelPort {
    attribute temperature : Temperature;
    out item fuelSupply : Fuel;     // directed features
    in  item fuelReturn : Fuel;
}

part def FuelTankAssembly { port fuelTankPort : FuelPort; }
part def Engine          { port engineFuelPort : ~FuelPort; }  // ~ = conjugated port
```

`~PortDef` is the **conjugate** port - it reverses the direction of all directed features. Two ports connect compatibly when their directed features match with inverse directions. Directed features are always referential, so `ref` is implicit on them.

### Connections and interfaces

```sysml
connection def PressureSeat {
    end bead : TireBead;
    end mountingRim : TireMountingRim;
}

interface def FuelInterface {           // an interface is a connection whose ends are ports
    end port supplierPort : FuelPort;
    end port consumerPort : ~FuelPort;
}

part vehicle : Vehicle {
    part tankAssy : FuelTankAssembly;
    part eng : Engine;
    interface : FuelInterface
        connect supplierPort ::> tankAssy.fuelTankPort
             to consumerPort ::> eng.engineFuelPort;
}
```

Use `connect a to b;` for a generic connection. Feature chains (`tankAssy.fuelTankPort`) navigate nested features ("dot notation," the successor to v1 property paths).

### Flows

```sysml
part def Vehicle {
    part fuelTank : FuelTank { out fuelOut : Fuel; }
    part engine   : Engine   { in  fuelIn  : Fuel; }
    flow fuelTank.fuelOut to engine.fuelIn;   // transfers Fuel along the connection
}
```

## Behavior

### Actions

```sysml
action def Focus { in scene : Scene;  out image : Image; }
action def Shoot { in image : Image;  out picture : Picture; }

action def TakePicture {
    in scene : Scene;
    out picture : Picture;

    action focus : Focus { in scene; out image; }
    action shoot : Shoot { in image; out picture; }

    flow focus.image to shoot.image;   // object flow between actions
    first focus then shoot;            // succession (ordering)
}
```

- `flow x to y;` transfers items between action outputs/inputs.
- `first a then b;` asserts `a` completes before `b` begins (succession).
- `succession flow x to y;` combines both: the output must be produced before it is consumed.

### States

```sysml
state def VehicleStates {
    in operatingVehicle : Vehicle;
    in controller : VehicleController;

    entry; then off;                  // initial transition into 'off'
    state off;

    accept VehicleStartSignal
        if operatingVehicle.brakePedalDepressed             // guard
        do send new ControllerStartSignal() to controller   // effect
        then starting;

    state starting;
    accept VehicleOnSignal then on;
    state on {
        entry performSelfTest;
        do action providePower;
        exit action applyParkingBrake;
    }
    accept VehicleOffSignal then off;
}
```

Each state may declare at most one `entry`, `do`, and `exit` action. Transitions use `accept <trigger> [if <guard>] [do <effect>] then <target>;`. Triggers can be:

- **Signal**: `accept SomeSignal`
- **Change**: `accept when expr > threshold`
- **Absolute time**: `accept at vehicle.maintenanceTime`
- **Relative time**: `accept after 48 [h]`

Add `parallel` to a state def for concurrent substates (no transitions allowed between concurrent substates).

### Calculations

```sysml
calc def StoppingDistance {
    in v : ISQ::SpeedValue;
    in a : ISQ::AccelerationValue;
    return : ISQ::LengthValue = v^2 / (2 * a);
}
```

## Requirements, Constraints, and Cases

### Constraints

```sysml
constraint def MassLimit {
    in massActual : ISQ::MassValue;
    in massMax : ISQ::MassValue;
    massActual <= massMax          // a Boolean expression
}
```

### Requirements (constraints with a subject)

A requirement is a **kind of constraint** evaluated against a **subject**, with optional **assumed** and **required** constraints. It is *satisfied* when, given true assumptions, all required constraints hold.

```sysml
requirement def MassRequirement {
    subject vehicle : Vehicle;
    attribute massRequired : ISQ::MassValue;

    assume constraint { vehicle.fluidMass <= 40 [kg] }
    require constraint { vehicle.mass <= massRequired }
}

requirement vehicleSpecification {
    subject v : Vehicle;
    requirement massReqt : MassRequirement;          // composite sub-requirement
    requirement fuelEconomyReqt : FuelEconomyRequirement;
}
```

Sub-requirements grouped by composition are treated as required constraints of the group and, by default, share the group's subject.

### Satisfy and verify

```sysml
part vehicle_c1 : Vehicle { /* ... */ }

satisfy vehicleSpecification by vehicle_c1;   // asserts the subject satisfies the requirement

verification case def MassVerification {
    subject vehicle : Vehicle;
    objective {
        verify requirement vehicleSpecification.massReqt;
    }
    // ... test/analysis actions producing a verdict
}
```

### Analysis cases

```sysml
analysis def FuelEconomyAnalysis {
    subject vehicle : Vehicle;
    objective fuelEconomyObjective {
        assume constraint { vehicle.wheelDiameter == 33 ['in'] }
        require constraint { fuelEconomyResult > 30 [mi / gallon] }
    }
    // steps are actions that compute the result
    return fuelEconomyResult : ISQ::DistancePerVolumeValue;
}
```

### Concerns and stakeholders

```sysml
part def 'Systems Engineer';
concern modularity {
    doc /* Parts should have well-defined interfaces understandable individually. */
    subject system;
    stakeholder se : 'Systems Engineer';
}
requirement r {
    frame concern modularity;   // a requirement that frames a concern
}
```

## Allocation

Allocation relates elements across viewpoints (e.g., a logical function to a physical component) using a connection-style `allocate … to …`.

```sysml
allocation def FunctionToComponent {
    end function : Action;
    end component : Part;
}

allocation processOrderAlloc : FunctionToComponent
    allocate ProcessOrder to OrderProcessor;
```

| Function / Behavior | Allocated To (Part) |
|---------------------|---------------------|
| ProcessOrder | OrderProcessor |
| ValidatePayment | PaymentGateway |
| ShipOrder | FulfillmentSystem |

## Additional Language Constructs

### Enumerations

An `enum def` is a kind of attribute definition limited to a fixed set of values. It cannot specialize another enumeration.

```sysml
enum def TrafficLightColor { red; yellow; green; }   // the 'enum' keyword is optional inside the body

part def TrafficLight {
    attribute currentColor : TrafficLightColor = TrafficLightColor::red;
}

// With units, by specializing a quantity:
enum def DiameterChoices :> ISQ::LengthValue {
    enum = 60 [mm];
    enum = 80 [mm];
    enum = 100 [mm];
}
```

### Use Cases

A `use case def` specifies a required interaction between a `subject` and one or more external `actor`s, with an `objective`. Actors are referential part usages. The subject must be declared before any actors.

```sysml
use case def 'Provide Transportation' {
    subject vehicle : Vehicle;
    actor driver : Person;
    actor passengers : Person[0..4];
    objective {
        doc /* Transport driver and passengers to the destination. */
    }
}

use case 'provide transportation' : 'Provide Transportation' {
    subject vehicle;
    first start;
    then include use case 'enter vehicle';
    then include use case 'drive vehicle';
    then include use case 'exit vehicle';
    then done;
}
```

`include use case` (or just `include`) performs another use case as a step; the included use case shares the container's subject.

### Variation and Variant Modeling

Mark any definition as a `variation` to express product-line variability; its `variant` usages are the allowed choices.

```sysml
variation part def EngineChoices :> Engine {
    variant '4cylEngine';
    variant '6cylEngine';
}

variation attribute def DiameterChoices :> Diameter {
    variant attribute diameterSmall = 70 [mm];
    variant attribute diameterLarge = 100 [mm];
}
```

(An `enum def` is implicitly a variation whose variants are its enumerated values.)

### Occurrences, Individuals, Time Slices, and Snapshots

This is the v2 "4D" model of things that exist over time. Parts, items, and actions are all `occurrence`s; attributes are not. Reach for it when feature values change across a lifetime.

```sysml
occurrence def Flight {
    ref part aircraft : Aircraft;
    timeslice preflight;       // a portion of the lifetime (nonzero duration)
    timeslice inflight;
    snapshot takeoff;          // an instant (zero-duration time slice); 'start' and 'done' are predefined
}

individual def Flight_248 :> Flight;   // a single, uniquely identified occurrence

individual part def Vehicle_1 :> Vehicle {
    snapshot part vehicle_1_t0 : Vehicle_1 { :>> mass = 2000.0; }
    then snapshot part vehicle_1_t1 : Vehicle_1 { :>> mass = 1500.0; }
}
```

`first a then b;` (or the `then` shorthand) orders occurrences in time. An `event occurrence` references something that happens during another occurrence; `message m of T from a.x to b.y;` asserts a transfer between them.

### Expressions and Feature Values

A feature value (`= expr`) binds a feature to a computed result. The expression language has arithmetic and comparison operators, feature chains, indexing (`x#(i)`), and collection operations such as `sum`, `size`, `->forAll`, and `->select`.

```sysml
part compositeThing : MassedThing {
    part subcomponents : MassedThing[*];
    attribute :>> totalMass = simpleMass + sum(subcomponents.totalMass);
}

constraint allPositive {
    in values : Real[*];
    values->forAll { in v : Real; v > 0 }
}
```

## Standard Library

SysML v2 ships a rich standard library. Prefer it over hand-rolled types:

- **ScalarValues** - `Real`, `Integer`, `Boolean`, `String`, `Natural`, …
- **ISQ / SI / Quantities / UnitsAndScales** - quantity values (`MassValue`, `ForceValue`, `LengthValue`, …) and units.
- **Items, Parts, Ports, Connections, Actions, States, Requirements, Cases, Metaobjects** - the base definitions/usages every element implicitly specializes.

```sysml
private import ScalarValues::*;
private import ISQ::*;
attribute efficiency : Real;
attribute power : PowerValue = 110 [kW];
```

## Language Extension (the v2 way - NOT UML profiles)

To extend SysML v2 for a domain you do **not** define stereotypes. You:

1. Create a **library package** of domain concepts (base definitions + base usages).
2. Define **semantic metadata** binding each concept to its base usage.
3. Optionally expose **user-defined keywords** (`#name`) for ergonomic authoring.

```sysml
library package FailureModeling {
    abstract occurrence def Failure { attribute severity : Level; }
    abstract occurrence failures : Failure[*] nonunique;
}

library package FailureModelingMetadata {
    private import FailureModeling::*;
    private import Metaobjects::SemanticMetadata;
    metadata def failure :> SemanticMetadata {
        :>> baseType = failures meta SysML::Usage;
    }
}

// Usage: the #failure keyword implicitly specializes the library concept
private import FailureModelingMetadata::*;
#failure 'device shutoff' {
    :>> severity = LevelEnum::high;
}
```

Plain (non-semantic) metadata simply tags elements so tooling can group, filter, or carry tool-specific data:

```sysml
metadata def Safety { attribute isMandatory : Boolean; }

part seatBelt[2] {
    @Safety { isMandatory = true; }   // @ is shorthand for the 'metadata' keyword
}

package 'Mandatory Safety Features' {
    public import vehicle::**[@Safety and Safety::isMandatory];   // metadata-filtered import
}
```

## Views, Viewpoints, and Graphical Notation

Graphical notation is a **projection** of the model. Define `viewpoint`s (stakeholder concerns) and `view`s (what to render and how), then render. The same parts/connections appear in an interconnection view, a tree view, a state-transition view, etc. - there is no separate "BDD file" vs "IBD file." Author the model in text; produce diagrams as views over it.

## Model Interchange and the Systems Modeling API

SysML v2 standardizes interoperability rather than relying on tool-specific XMI:

- A standardized **Systems Modeling API and Services** (REST/HTTP and alternatives) for projects, commits, branches, queries, and element CRUD.
- A defined **JSON representation** of the (KerML/SysML) abstract syntax for interchange.

This is the integration surface for portability, programmatic generation, diffing, and tool integration - not the textual file format.

## MBSE Workflow

1. **Requirements** - capture needs and system requirements as evaluable constraints with subjects.
2. **Structure** - define parts, attributes, items; compose the system hierarchy.
3. **Interfaces** - define ports/interfaces; connect parts and specify flows.
4. **Behavior** - actions, states, sequences; tie behavior to structure (`perform`, `exhibit`).
5. **Constraints & analysis** - parametric constraints and analysis cases for physics/performance.
6. **Allocate** - map functions/behavior to structural components.
7. **Verify** - verification cases that `verify` requirements; assert `satisfy` against designs.
8. **Trace** - definitions/usages, satisfy, derive, refine, and metadata give end-to-end traceability in one model.

## End-to-End Example

A minimal but complete model showing how the pieces connect: a requirement with a subject and a constraint, a structural decomposition with a port and an internal flow, a behavior tied to structure with `perform`, and a `satisfy` assertion. This is an original illustrative model; for full real-world models see the resources below.

```sysml
package CoffeeMaker {
    private import ISQ::*;
    private import ScalarValues::*;

    // --- Requirement (a constraint on a subject) ---
    requirement def BrewTimeRequirement {
        subject machine : CoffeeMachine;
        attribute maxBrewTime : TimeValue;
        require constraint { machine.brewTime <= maxBrewTime }
    }

    // --- Structure ---
    item def Water;
    item def Coffee;

    port def WaterPort { in item water : Water; }

    part def CoffeeMachine {
        attribute brewTime : TimeValue;
        port intake : WaterPort;
        part boiler : Boiler;
        part brewUnit : BrewUnit;
        flow boiler.hotWater to brewUnit.waterIn;   // internal item flow
        perform action brew : Brew;                 // behavior tied to this structure
    }

    part def Boiler   { out hotWater : Water; }
    part def BrewUnit { in waterIn : Water; out coffee : Coffee; }

    // --- Behavior ---
    action def Brew { in water : Water; out coffee : Coffee; }

    // --- A concrete machine and the satisfaction assertion ---
    part myMachine : CoffeeMachine {
        attribute :>> brewTime = 45 [s];
    }

    requirement brewReq : BrewTimeRequirement {
        attribute :>> maxBrewTime = 60 [s];
    }

    satisfy brewReq by myMachine;   // binds myMachine to the requirement's subject
}
```

## Best Practices

### Model organization - organize by `package`, not by diagram type

```text
Model/
├── 1_Requirements.sysml      // requirement defs/usages, concerns, MOEs
├── 2_Structure.sysml         // part/item/attribute defs and the system tree
├── 3_Interfaces.sysml        // port/interface/connection defs, flows
├── 4_Behavior.sysml          // action/state/calc defs
├── 5_Analysis.sysml          // constraint defs, analysis & verification cases
├── 6_Allocation.sysml        // allocation defs/usages
└── lib/
    └── DomainExtensions.sysml // library package(s) + semantic metadata
```

### Naming conventions

| Element | Convention | Example |
|---------|------------|---------|
| Definition | PascalCase noun | `VehicleSystem`, `FuelPort` |
| Usage | camelCase noun | `engine`, `fuelTankPort` |
| Directed feature | camelCase, direction declared | `in scene`, `out picture` |
| Requirement def | PascalCase + "Requirement" | `MassRequirement` |
| Human ID | short name in `< >` | `requirement <'REQ-001'> systemPerformance` |

Quote names containing spaces or punctuation: `part def 'Front Axle';`.

## Common v1 → v2 Pitfalls (anti-patterns to avoid)

- **Do not** use PlantUML or any drawing DSL as "the model." Author the SysML v2 textual notation; diagrams are views.
- **Do not** write stereotypes (`<<block>>`, `<<requirement>>`). Use native keywords.
- **Do not** think in BDD vs IBD. There is one model.
- **Do not** say "block." Use `part def` / `part`.
- **Do not** model requirements as text-only boxes. Express the testable condition as `require constraint { ... }` with a `subject`.
- **Do not** invent value types when ISQ/SI quantities exist.
- **Do not** extend the language with profiles. Use library packages + metadata + user-defined keywords.

## Authoritative Resources and References

Specifications (the normative sources):
- OMG **Systems Modeling Language (SysML) v2** specification (Language; v1-to-v2 transformation guidance).
- OMG **Kernel Modeling Language (KerML)** specification, the semantic foundation.
- OMG **Systems Modeling API and Services** specification, for interoperability and JSON / abstract-syntax interchange.
- OMG **Introduction to the SysML v2 Language**, Textual and Graphical Notation (release 2026-01).

Specification source, grammars, and standard library:
- SysML v2 monthly release (BNF grammars, standard libraries, training, example models): https://github.com/Systems-Modeling/SysML-v2-Release

Reference implementation and tooling:
- Pilot implementation (textual parser, Jupyter kernel, validation): https://github.com/Systems-Modeling/SysML-v2-Pilot-Implementation

API and integration:
- API and Services reference implementation: https://github.com/Systems-Modeling/SysML-v2-API-Services
- API Cookbook (worked recipes): https://github.com/Systems-Modeling/SysML-v2-API-Cookbook
- Python API client: https://github.com/Systems-Modeling/SysML-v2-API-Python-Client
- Java API client: https://github.com/Systems-Modeling/SysML-v2-API-Java-Client

Example models to study:
- Airbus Apollo 11 model: https://github.com/airbus/apollo-11-sysml-v2
- GfSE SysML v2 models: https://github.com/GfSE/SysML-v2-Models

Domain and method extensions:
- AADL library and release: https://github.com/Systems-Modeling/SysML-v2-AADL-Release
- Functional Architecture for Systems (FAS) method: https://github.com/GfSE/fas4sysmlv2
- Systems Architecture Framework (SAF): https://github.com/GfSE/SAF-SysMLV2

## License & Attribution

This reference is released under **CC BY 4.0** - free to share and adapt with attribution. (To use a code-oriented license instead, replace the `license` field above with `MIT` and add a `LICENSE` file.)

SysML v2 and KerML are standards of the Object Management Group (OMG). Code examples follow the OMG SysML v2 / KerML specifications and the OMG "Introduction to the SysML v2 Language." SysML® is a registered trademark of OMG. This document is an independent reference and is not an OMG publication.

> Notation tracks the SysML v2 release train; for release-sensitive constructs, verify against the current OMG specification and monthly release.

---

**Skill version:** 1.1.0 · **Spec baseline:** SysML v2.0 / 2026-01 · **Last updated:** 2026-06-07
