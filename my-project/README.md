# Hypersonic Vehicle Design Ontology

This project models a hypersonic vehicle design process as an ontology using [OML](https://www.modelware.io/). The purpose is to represent the main engineering subsystems, their parts, interfaces, requirements, and design tradeoffs that influence vehicle performance and range.

The ontology is organized around the major design concerns for a hypersonic platform:

- Aerodynamics and Structures
- Thermal System
- Propulsion
- Stability and Control
- Payload

These domains are represented as subsystems, parts, ports, connections, and requirement traceability so the design can be reasoned about as a connected system rather than as disconnected engineering silos.

## System being modeled

This model supports a conceptual design process for a hypersonic vehicle. It captures how the vehicle is decomposed into subsystems and parts, how those parts exchange information or energy through ports, and how engineering decisions influence one another.

Examples of the design relationships captured here include:

- wings, nacelle, body, nose cone, tail, and stabilizing surfaces in the aerodynamics and structures domain
- thermal stackups for nose, leading edge, nacelle, and body regions
- propulsion subsystem behavior based on engine mass and thrust
- stability and control elements such as actuators and control unit logic
- payload and fuel mass as part of mission-level mass and range implications
- requirement-to-concern-to-capability traceability for the engineering design process

The model is intentionally structured to support both engineering decomposition and design reasoning. It does not just describe a single component; it describes how system elements interact and constrain one another across the vehicle.

## Project structure

The repository is organized into vocabulary and model layers.

### Core vocabulary

- `src/method/oml/www.example.com/method/vocabulary.oml`
  - Defines the domain vocabulary, concepts, aspects, properties, relations, and rules.
  - This is the main place to define the language of the system.

### Base containment vocabulary

- `src/method/oml/www.omg.org/spec/Commons/Base.oml`
  - Provides the `base:isContainedBy` relation used for part-to-subsystem containment.

### Description / instance model

- `src/model/oml/www.example.com/project/description.oml`
  - Stores the actual subsystem and part instances, ports, connections, and relationships.
  - This is the concrete model of the vehicle architecture.

### Supporting project files

- `src/model/oml/www.example.com/project/stakeholders.oml`
  - Stakeholder definitions used in requirement traceability.

- `src/model/oml/www.example.com/project/requirements.oml`
  - Requirement, concern, and capability definitions tied back to stakeholders.

## How to use this project

### 1. Understand the vocabulary layer

Start with the vocabulary file. This file defines the reusable ontology terms such as:

- `Subsystem`
- `Component`
- `Port`
- `Connection`
- `Requirement`
- `Stakeholder`
- `Concern`
- `Capability`
- domain-specific concepts such as `Wing`, `Engine`, `ThermalStackup`, and `StabilizingSurface`

If you want to add a new engineering concept or design property, add it here first.

### 2. Instantiate the vehicle in the description model

The description file is where the actual system is represented. It includes:

- subsystem instances such as `Sys_Aero`, `Sys_Propulsion`, and `Sys_Thermal`
- part instances such as `Part_Wings`, `Part_Engine`, and `Part_Fuel`
- port instances and directions
- connection instances between ports
- containment relationships between parts and subsystems

This is where the design becomes concrete.

### 3. Validate the model

From the project root, use:

- `oml lint` to check for syntax and structure issues
- `oml reason` to check logical consistency and inferred relationships

These commands should be run after making any changes to the vocabulary or instance model.

### 4. Extend the ontology deliberately

When adding new domain items, follow this pattern:

1. Add the new concept or property to the vocabulary file.
2. Add or update instance assertions in the description file.
3. Keep relation semantics consistent with the rest of the model.
4. Validate with `oml lint` and `oml reason`.

## Typical engineering usage

This ontology is useful for asking questions such as:

- Which subsystems are connected to each other?
- Which parts are contained within each subsystem?
- Which ports are used to exchange mass, flow, or control information?
- Which requirements are tied to which concerns and capabilities?
- How do subsystem dependencies affect range, mass, propulsion, and thermal performance?

It is intended as a conceptual system model for early-stage design reasoning, especially when analyzing tradeoffs between vehicle performance, mass, thermal constraints, and control effects.

## Quick start

1. Open the project in VS Code with OML support.
2. Review the vocabulary in `src/method/oml/www.example.com/method/vocabulary.oml`.
3. Inspect the concrete vehicle model in `src/model/oml/www.example.com/project/description.oml`.
4. Run `oml lint`.
5. Run `oml reason`.
6. Iterate on the ontology as the design matures.

## Commands

- `oml lint` — checks OML files for structural issues
- `oml reason` — checks ontology consistency and logical inference
- `oml export -o build/owl` — exports the ontology to OWL if needed

This project is intended to serve as a reusable ontology for hypersonic vehicle design reasoning and tradespace analysis.
