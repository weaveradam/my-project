# OML Method for Hypersonic Vehicle Design

## What the method prescribes

This method prescribes four reusable description patterns:

1. **System decomposition**: represent a vehicle as subsystems containing physical
   parts, and use containment consistently rather than encoding hierarchy in names.
2. **Interface and connection**: represent exchanges through ports and typed
   connections, with port direction and transfer semantics recorded at the edge.
3. **Requirement traceability**: connect each requirement to its stakeholder,
   concern, and enabled capability.
4. **Analysis and verification**: represent computational analyses and physical
   tests as first-class tasks that inform or verify subsystems.

The method separates vocabulary from descriptions. The vocabulary defines terms
and relation characteristics; the editors and templates describe how a modeler
should create instances; the project description is the resulting model.

## Why these patterns

The four patterns cover the minimum reasoning loop for an early vehicle design:
structure establishes scope, interfaces expose coupling, requirements explain
why a decision matters, and evidence records how confidence is earned. A single
component-only pattern was rejected because it would not show system tradeoffs.
An executable optimization pattern was also deferred: the current vocabulary
does not define units, solver provenance, or result uncertainty, so pretending
that computed values are authoritative would overstate the model.

## Business rules

The following rules are method obligations. Editors should reject or flag a
description that violates them:

1. Every `Subsystem` description must have at least one contained
   `EngineeringArtifact`; containment should be asserted with `contains` or
   `base:isContainedBy`, not duplicated with an unrelated naming convention.
2. Every `ThermalSubsystem` must contain at least one `ThermalStackup`, and each
   thermal stackup must be contained by the physical region it protects.
3. A `Connection` must have exactly one source port and one destination port.
   The source and destination must belong to different components, and the
   source direction must be `out` while the destination direction is `in`.
4. Every `Requirement` must have exactly one stakeholder, at least one concern,
   and at least one enabled capability. The functional `isStatedBy` relation
   enforces the single-stakeholder part in the vocabulary.
5. Every analysis or test must either `informs` or `verifies` at least one
   subsystem. A verification claim without a target is not evidence.
6. When `A constrains B` and `B constrains C`, the vocabulary rule derives
   `A indirectlyConstrains C`. This is useful for trade studies but must not be
   presented as a direct causal claim.
7. The simplified reliability screen evaluates every physical part as
   `marginOfSafety - standardDeviation >= 1`, then reports the passing-part
   fraction. It must be labeled a screening score, not a calibrated probability.
   Range analysis links relevant physical parts to an analysis task through
   `SimulationTask.includesRangeRelevantPart`.

Rules 1–5 are validation/editor rules because the current OML rule fragment is
best suited to inference rather than closed-world cardinality validation. Rule 6
is implemented directly as `IndirectConstraint` in the vocabulary. Rule 7 is
computed in the analysis layer from the explicit illustrative assumptions in the
model.

## Uncertainty and open issues

The margin-based score assumes a standard deviation of 0.3 for every part and
does not model independence or failure distributions. It is not a calibrated
reliability probability. Range inputs, including TSFC, cruise speed, L/D, fuel
capacity, and thermal-stackup fuel-capacity displacement, are illustrative. In
particular, fuel-capacity displacement uses kg-equivalent estimates rather than
geometric volume, and the Breguet calculation assumes steady powered cruise.
These numbers are examples for analysis, not design-authoritative estimates.
Port compatibility is currently expressed by direction and transfer text; a
future revision should add an explicit `InterfaceType` concept. Analysis
provenance, solver version, and test results remain open issues.

## Layout

- `src/method/oml/.../vocabulary.oml`: shared concepts, relations, and inference.
- `src/method/patterns/`: rationale, alternatives, uncertainty, and open issues
  for each pattern.
- `src/method/editors/`: one editor specification per pattern.
- `src/method/templates/`: reusable `.oml.template` Compose templates consumed
  by the editors. The template extension keeps placeholder text out of the OML
  source scanner until an editor expands it into a concrete description.
- `src/model/oml/.../description.oml`: dogfooded vehicle instances.
- `notebooks/method_dogfooding.ipynb`: narrative and live OML snippets for ten
  representative instances.
- `src/method/md/.../analysis/dashboard.md`: reusable Compose dashboard with
  graph, matrix, and chart views.
- `src/model/md/.../analysis/`: project dashboard invocation and pattern-gap
  queries; the dashboard includes the script-computed direct-tie analysis.
- `ANALYSIS.md`: Module 1 question, evidence, finding, and diagnosed-gap record.
