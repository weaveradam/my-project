# Interface and Connection Editor

**Inputs:** source component and port, destination component and port, direction
for each port, and transfer description.

**Checks:** source is `out`, destination is `in`, endpoints are different
components, and the transfer description is non-empty.

**Compose output:** use `templates/connection.oml.template`; use
`templates/port.oml.template` for any new ports and
`templates/has-port.oml.template` to attach
them.
