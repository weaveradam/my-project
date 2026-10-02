# Pattern: Interface and Connection

## Intent

Make exchanges between components explicit through ports and connections.

## Rationale and alternatives

Ports preserve the boundary of a component and allow direction to be checked.
Direct component-to-component relations were rejected because they lose the
exchange endpoint and cannot describe multiple flows between the same pair.

## OML shape

Create `Port` instances, attach them with `hasPort`, then create a `Connection`
relation with `from`, `to`, and a concise `transfers` description.

## Uncertainty and open issue

Transfer text is currently free-form. A future `InterfaceType` taxonomy should
replace text when compatibility, units, or conservation laws become important.
