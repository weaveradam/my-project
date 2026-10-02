---
ontology: http://www.example.com/project/description.bundle#
---
# Pattern Gap Queries

These orphan queries use absence intentionally: each returns a concrete
element whose expected relationship is not present. An empty result is a clean
pattern check; it does not prove unmodeled facts are true.

## Conformance — thermal subsystems contain thermal stackups

This query returns violations. The current model should return no rows.

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?subsystem
WHERE {
  ?subsystem a method:ThermalSystem .
  FILTER NOT EXISTS {
    ?subsystem method:contains ?stackup .
    ?stackup a method:ThermalStackup .
  }
}
ORDER BY ?subsystem
```

## Near miss — informed subsystems without a verifying test

An analysis that informs a subsystem but has no physical test verifying that
subsystem is a near miss for evidence completeness, not a failed design.

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT DISTINCT ?analysis ?subsystem
WHERE {
  ?analysis a method:ComputationalAnalysis ;
    method:informs ?subsystem .
  FILTER NOT EXISTS {
    ?test a method:PhysicalTest ;
      method:verifies ?subsystem .
  }
}
ORDER BY ?subsystem ?analysis
```

## System decomposition — uncontained physical parts

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>
PREFIX base: <http://www.omg.org/spec/Commons/Base#>

SELECT ?part
WHERE {
  VALUES ?partType {
    method:Wing method:Nacelle method:Body method:NoseCone method:Tail
    method:StabilizingSurface method:ThermalStackup method:NoseStackup
    method:LeadingEdgeStackup method:NacelleStackup method:BodyStackup
    method:Engine method:ControlSurfaceActuator method:ControlUnit
    method:MissionPayload method:Fuel
  }
  ?part a ?partType .
  FILTER NOT EXISTS { ?part base:isContainedBy ?parent }
  FILTER NOT EXISTS { ?parent method:contains ?part }
}
ORDER BY ?part
```

## Interface and connection — ports with missing ownership or use

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>
PREFIX oml: <http://opencaesar.io/oml#>

SELECT DISTINCT ?port ?gap
WHERE {
  ?port a method:Port .
  {
    FILTER NOT EXISTS { ?component method:hasPort ?port }
    BIND("port has no owning component" AS ?gap)
  }
  UNION
  {
    FILTER NOT EXISTS {
      ?connection a method:Connection ;
        oml:hasSource ?port .
    }
    FILTER NOT EXISTS {
      ?connection a method:Connection ;
        oml:hasTarget ?port .
    }
    BIND("port is not used by a connection" AS ?gap)
  }
}
ORDER BY ?port ?gap
```

## Requirement traceability — missing stakeholder, concern, or capability

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?requirement ?gap
WHERE {
  ?requirement a method:Requirement .
  {
    FILTER NOT EXISTS { ?requirement method:isStatedBy ?stakeholder }
    BIND("missing stakeholder" AS ?gap)
  }
  UNION
  {
    FILTER NOT EXISTS { ?requirement method:addressesConcern ?concern }
    BIND("missing concern" AS ?gap)
  }
  UNION
  {
    FILTER NOT EXISTS { ?requirement method:enablesCapability ?capability }
    BIND("missing capability" AS ?gap)
  }
}
ORDER BY ?requirement ?gap
```

## Analysis and verification — tasks without a subsystem target

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?task
WHERE {
  VALUES ?taskType { method:ComputationalAnalysis method:PhysicalTest }
  ?task a ?taskType .
  FILTER NOT EXISTS { ?task method:informs ?subsystem }
  FILTER NOT EXISTS { ?task method:verifies ?subsystem }
}
ORDER BY ?task
```

## Range analysis — relevant physical parts missing from the analysis

```sparql
PREFIX method: <http://www.example.com/method/vocabulary#>

SELECT ?part
WHERE {
  VALUES ?componentType {
    method:Wing method:Nacelle method:Body method:NoseCone method:Tail
    method:StabilizingSurface method:NoseStackup method:LeadingEdgeStackup
    method:NacelleStackup method:BodyStackup method:Engine
    method:ControlSurfaceActuator method:ControlUnit method:MissionPayload
    method:Fuel
  }
  ?part a ?componentType .
  FILTER NOT EXISTS {
    <http://www.example.com/project/description#Task_RangeSensitivityEstimate>
      method:includesRangeRelevantPart ?part
  }
}
ORDER BY ?part
```
