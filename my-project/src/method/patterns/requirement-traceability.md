# Pattern: Requirement Traceability

## Intent

Preserve the reason for a design obligation from stakeholder through concern to
capability.

## Rationale and alternatives

The three-link path supports impact analysis without pretending that a requirement
is itself a subsystem. A requirement-only catalog was rejected because it loses
motivation; embedding stakeholder text in comments was rejected because it is not
queryable.

## OML shape

Create a `Requirement`, link it with `isStatedBy`, `addressesConcern`, and
`enablesCapability`.

When a question asks which physical parts are affected by a concern such as
range sensitivity, record the parts examined by a dedicated analysis using
`SimulationTask.includesRangeRelevantPart`. A requirement-to-concern link alone
does not establish physical-part impact.

## Uncertainty and open issue

Priority, verification method, and acceptance threshold are not yet modeled.
Those should be added before using the pattern for contractual requirements.
The range-sensitivity requirement is supported by a range-analysis task that
links to the physical parts included in the calculation. The current OML
description organization does not link that task directly to the requirement
individual in the separate requirements description. These links express the
chosen equation and assumptions; they do not encode sensitivity magnitude or
validated causality.
