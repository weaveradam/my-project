# Analysis Findings

This analysis answers the three Module 1 questions using asserted project
evidence. A missing answer is reported as a diagnosed gap rather than filled
with an assumed relationship.

## 1. Reliability variables for program officers

**Question:** What variables can be considered and translated across systems to
enhance a reliability estimate for program officers?

**Evidence:** The description now assigns margin of safety and a standard
deviation to all 15 physical parts. The model uses the requested pass/fail
transformation, `marginOfSafety - standardDeviation >= 1`, and averages the
resulting binary outcomes.

**Finding — answered under an explicitly rudimentary model:** Fourteen parts
pass and `Part_Body` fails (`1.2 - 0.3 = 0.9`), so the margin-based score is
`14 / 15 = 0.9333`. The standard deviation of `0.3` is a made-up illustrative
input as requested. This value is a screening pass fraction, not a statistically
validated probability of vehicle reliability; component independence and
failure distributions are not modeled.

## 2. Direct subsystem ties for department heads

**Question:** Which subsystems are directly correlated, and does one subsystem
have more direct ties than others?

**Evidence:** The graph and scripted degree computation query asserted
`method:constrains` relationships from the system description. Seven directed
edges are present:

| Source | Directly constrains |
|---|---|
| `Sys_Aero` | `Sys_Thermal`, `Sys_Propulsion` |
| `Sys_Thermal` | `Sys_Propulsion` |
| `Sys_Propulsion` | `Sys_Payload` |
| `Sys_Stability` | `Sys_Aero`, `Sys_Payload` |
| `Sys_Payload` | `Sys_Thermal` |

The script counts both incoming and outgoing direct ties:

| Subsystem | Outgoing | Incoming | Total direct ties |
|---|---:|---:|---:|
| `Sys_Aero` | 2 | 1 | 3 |
| `Sys_Thermal` | 1 | 2 | 3 |
| `Sys_Propulsion` | 1 | 2 | 3 |
| `Sys_Payload` | 1 | 2 | 3 |
| `Sys_Stability` | 2 | 0 | 2 |

**Finding — answered:** Four subsystems tie for the largest number of direct
ties: Aero, Thermal, Propulsion, and Payload. Stability has two. This is a
structural constraint count, not a measured statistical correlation or a ranking
of causal importance.

## 3. Components affected by range changes

**Question:** What components are meaningfully affected by changes in the
missile's range?

**Evidence:** `Req_RangeSensitivity` addresses `Concern_RangeSensitivity`. The
`Task_RangeSensitivityEstimate` analysis references all 15 modeled physical
parts through `includesRangeRelevantPart`. The calculation includes every
non-fuel part in dry mass, the wing's lift-to-drag ratio, cruise speed, engine
TSFC, fuel load, and fuel capacity reduced by thermal-stackup displacement.

**Finding — answered with illustrative assumptions:** Non-fuel dry mass is
`2304 kg`. The four thermal stackups displace `80 kg-equivalent` from a nominal
`260 kg` fuel capacity, leaving `180 kg` usable fuel (less than the modeled
`220 kg` fuel load). Using wing L/D `6.0`, cruise speed `1500 m/s`, TSFC
`0.000018 kg/(N·s)`, and `g = 9.80665 m/s²`, the Breguet jet-range estimate is
approximately `3835.3 km`. Payload mass and every other part's mass contribute
to dry mass; thermal stackups additionally reduce available fuel capacity.
Capacity displacement and propulsion estimates are illustrative, not validated
engineering data.

## Pattern gap detection

The accompanying analysis pages demonstrate conformance, near-miss, orphan,
coverage, and constructed-graph queries. The gap-query page provides one
orphan/missing-relationship query for each original pattern:

- **System decomposition:** physical parts without a containment relationship.
- **Interface and connection:** ports without ownership or connection use.
- **Requirement traceability:** requirements missing stakeholder, concern, or
  capability links.
- **Analysis and verification:** evidence tasks without subsystem targets.

The range-analysis orphan query checks for physical parts omitted from the
analysis. Its part links are assumptions tied to the implemented range formula;
they are not independent proof that each part has equal sensitivity. OML
descriptions cannot reference other descriptions directly, so the analysis task
is not formally linked to the requirement individual in the separate requirements
module. Orphaned ports and other missing links remain findings to resolve.

## Dashboard and computation

The composed dashboard has three rendered view types: a graph of constraints, a
subsystem-by-evidence matrix with explicit zeros, and a bar chart of direct ties.
Its Python block performs query → computation → table rendering for direct tie
counts. The reusable dashboard template is defined under the method namespace
and invoked by the project dashboard page.
