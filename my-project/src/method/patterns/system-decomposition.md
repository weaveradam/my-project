# Pattern: System Decomposition

## Intent

Represent the vehicle as a hierarchy of subsystems and contained artifacts.

## Rationale and alternatives

Containment makes ownership and scope queryable. A flat list of components was
rejected because it obscures subsystem tradeoffs; encoding hierarchy only in
instance names was rejected because names cannot support transitive reasoning.

## OML shape

Use `Subsystem`, a domain specialization such as `PropulsionSystem`, and
`contains`/`base:isContainedBy` for the parent-child assertion.

## Uncertainty and open issue

The method does not yet distinguish physical containment from logical allocation.
That distinction should be added if the project begins modeling software or
cross-cutting functions.
