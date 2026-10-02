# Analysis and Verification Editor

**Inputs:** task name, task type (`ComputationalAnalysis` or `PhysicalTest`),
target subsystem, relationship (`informs` or `verifies`), and scope note.

**Checks:** task type is valid, target is a `Subsystem`, and at least one
evidence relationship is selected.

**Compose output:** use `templates/evidence-task.oml.template`.
