# Pattern: Analysis and Verification

## Intent

Record which analyses inform design decisions and which physical tests verify
subsystems.

## Rationale and alternatives

Treating evidence as an instance keeps the design rationale connected to the
system model. A prose-only test plan was rejected because it cannot be queried;
making every analysis a subsystem was rejected because an analysis is an activity,
not a product boundary.

## OML shape

Use `ComputationalAnalysis` or `PhysicalTest`, then assert `informs` and/or
`verifies` to the affected subsystem.

## Uncertainty and open issue

The current model does not capture result values, confidence, provenance, or
pass/fail status. The pattern therefore records evidence relationships, not
certification.
