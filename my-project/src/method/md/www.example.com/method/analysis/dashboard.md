---
template:
  id: http://www.example.com/method/analysis/dashboard
  name: "Hypersonic Vehicle Analysis Dashboard"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Hypersonic Vehicle Analysis

This dashboard distinguishes modeled relationships from plausible engineering
hypotheses. The constraint network answers the department-head question about
direct subsystem ties. The reliability-variable matrix inventories candidate
inputs, not a reliability score. Range-impact candidates are deliberately
reported as a gap until the model contains supported impact assertions.

## Question 1 — Reliability inputs for program officers

The model records margin of safety for every physical part and uses an
illustrative standard deviation of 0.3 for the requested pass/fail estimate.
Other cross-system variables include mass, wing lift-to-drag ratio, engine
thrust, fuel capacity, and thermal-stackup capacity displacement. The reliability
score is a pass fraction under the chosen rule, not a statistically calibrated
probability of vehicle reliability.

```matrix
---
rowColumnLabel: Subsystem / Candidate reliability input
stylesheet:
  - selector: cell [Number(value) > 0]
    style:
      background-color: lightyellow
---
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?row ?column (MAX(?present) AS ?value)
WHERE {
  VALUES ?row {
    <http://www.example.com/project/description#Sys_Aero>
    <http://www.example.com/project/description#Sys_Thermal>
    <http://www.example.com/project/description#Sys_Propulsion>
    <http://www.example.com/project/description#Sys_Stability>
    <http://www.example.com/project/description#Sys_Payload>
  }
  VALUES (?column ?property) {
    ("massKg" method:massKg)
    ("marginOfSafety" method:marginOfSafety)
    ("standardDeviation" method:standardDeviation)
    ("lift" method:lift)
    ("drag" method:drag)
    ("thrust" method:thrust)
    ("conductionCoefficient" method:conductionCoefficient)
    ("maxInternalTempF" method:maxInternalTempF)
    ("targetLiftToDrag" method:targetLiftToDrag)
    ("liftToDragRatio" method:liftToDragRatio)
  }
  OPTIONAL {
    { ?row ?property ?raw }
    UNION
    { ?row method:contains ?part . ?part ?property ?raw }
  }
  BIND(IF(BOUND(?raw), 1, 0) AS ?present)
}
GROUP BY ?row ?column
ORDER BY ?row ?column
```

**Finding:** The requested rudimentary reliability score is now computable. For
each physical part, calculate `marginOfSafety - standardDeviation`; a result
below 1 is a failure (0), otherwise a success (1). The arithmetic mean of those
indicators is the score. The `0.3` deviation is an illustrative assumption, not
measured scatter. This score answers the project question under its stated
simplified rule; it is not a true probability of system reliability.

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?subsystem
WHERE {
  ?subsystem a method:Subsystem .
  FILTER NOT EXISTS {
    ?subsystem method:hasReliabilityEstimate ?estimate
  }
}
ORDER BY ?subsystem
```

## Question 2 — Direct subsystem ties for department heads

The graph shows only asserted direct `constrains` relations. Indirectly inferred
relationships are intentionally excluded so direct ties are not confused with
transitive impact.

```graph
---
layout:
  mode: force
  running: true
  fit: true
  padding: 24
  force:
    repulsion: 2500
    linkDistance: 150
stylesheet:
  - selector: node
    style:
      fill: lightblue
      stroke: grey
      stroke-width: 1
---
PREFIX method: <http://www.example.com/method/vocabulary#>
PREFIX diagram: <http://opencaesar.io/diagram#>

CONSTRUCT {
  ?source a diagram:Node ;
    diagram:text ?sourceLabel ;
    diagram:class "subsystem" .
  ?target a diagram:Node ;
    diagram:text ?targetLabel ;
    diagram:class "subsystem" .
  ?edge a diagram:Edge ;
    diagram:source ?source ;
    diagram:target ?target ;
    diagram:text "constrains" .
}
WHERE {
  ?source method:constrains ?target .
  BIND(REPLACE(STR(?source), "^.*[#/]", "") AS ?sourceLabel)
  BIND(REPLACE(STR(?target), "^.*[#/]", "") AS ?targetLabel)
  BIND(IRI(CONCAT("/constraint/", MD5(CONCAT(STR(?source), STR(?target))))) AS ?edge)
}
```

```chart
---
type: bar
data:
  labels: subsystemLabel
  datasets:
    - label: Direct ties (in + out)
      data: directTies
options:
  indexAxis: y
  plugins:
    title:
      display: true
      text: Direct subsystem ties
    legend:
      display: false
  scales:
    x:
      beginAtZero: true
      ticks:
        precision: 0
---
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?subsystemLabel (COUNT(DISTINCT ?neighbor) AS ?directTies)
WHERE {
  VALUES ?subsystem {
    <http://www.example.com/project/description#Sys_Aero>
    <http://www.example.com/project/description#Sys_Thermal>
    <http://www.example.com/project/description#Sys_Propulsion>
    <http://www.example.com/project/description#Sys_Stability>
    <http://www.example.com/project/description#Sys_Payload>
  }
  OPTIONAL {
    { ?subsystem method:constrains ?neighbor }
    UNION
    { ?neighbor method:constrains ?subsystem }
    FILTER(?neighbor != ?subsystem)
  }
  BIND(REPLACE(STR(?subsystem), "^.*[#/]", "") AS ?subsystemLabel)
}
GROUP BY ?subsystem ?subsystemLabel
ORDER BY DESC(?directTies) ?subsystemLabel
```

**Finding:** The model asserts seven direct constraint edges. Counting one tie
for each direct incoming or outgoing neighbor gives 3 ties each for
`Sys_Aero`, `Sys_Thermal`, `Sys_Propulsion`, and `Sys_Payload`, and 2 for
`Sys_Stability`. There is no unique highest-degree subsystem. This is structural
coupling, not evidence of measured correlation or causal strength.

## Question 3 — Components affected by range changes

`Concern_RangeSensitivity` is linked to `Req_RangeSensitivity`. The
`Task_RangeSensitivityEstimate` analysis identifies all modeled physical parts
through `includesRangeRelevantPart`. Range is calculated from wing L/D, cruise
speed, engine TSFC, dry vehicle mass, and fuel load. Thermal stackups increase
dry mass and displace fuel capacity; payload mass increases dry mass; the weight
of every part enters the vehicle mass sum.

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT DISTINCT ?component ?property ?value
WHERE {
  <http://www.example.com/project/description#Task_RangeSensitivityEstimate>
    method:includesRangeRelevantPart ?component .
  ?component method:massKg ?value .
  BIND("massKg" AS ?property)
}
ORDER BY ?component ?property
```

**Finding:** All fifteen modeled physical parts, including payload, fuel,
engine, and thermal stackups, are traced as range-relevant. The calculation
reports an illustrative range estimate and exposes the assumed fuel-capacity
displacement so thermal protection can affect both dry mass and fuel load.
The Breguet estimate assumes steady powered cruise and the specific units
documented with the computed analysis.

## Analysis-evidence coverage

This coverage matrix intentionally includes zeros: absence of an informing or
verifying task is visible rather than silently omitted.

```matrix
---
rowColumnLabel: Subsystem / Evidence task
stylesheet:
  - selector: cell [Number(value) === 0]
    style:
      background-color: mistyrose
  - selector: cell [Number(value) > 0]
    style:
      background-color: lightgreen
---
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  VALUES ?row {
    <http://www.example.com/project/description#Sys_Aero>
    <http://www.example.com/project/description#Sys_Thermal>
    <http://www.example.com/project/description#Sys_Propulsion>
    <http://www.example.com/project/description#Sys_Stability>
    <http://www.example.com/project/description#Sys_Payload>
  }
  VALUES ?column {
    <http://www.example.com/project/description#Task_Gmsh_GridGeneration>
    <http://www.example.com/project/description#Task_SU2_HypersonicCFD>
    <http://www.example.com/project/description#Task_TaylorMaccoll_ODE>
    <http://www.example.com/project/description#Task_ThermalFEA>
    <http://www.example.com/project/description#Task_WindTunnel>
  }
  OPTIONAL {
    SELECT ?row ?column (COUNT(*) AS ?n)
    WHERE {
      {
        ?column method:informs ?row
      }
      UNION
      {
        ?column method:verifies ?row
      }
    }
    GROUP BY ?row ?column
  }
}
ORDER BY ?row ?column
```
