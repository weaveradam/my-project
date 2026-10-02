# System Decomposition Editor

**Inputs:** subsystem name, subsystem type, contained artifact names, rationale.

**Checks:** subsystem type is a `Subsystem`; at least one artifact is supplied;
each artifact has one intended owner.

**Compose output:** use `templates/subsystem.oml.template` once, then add one
`base:isContainedBy` assertion per artifact.
