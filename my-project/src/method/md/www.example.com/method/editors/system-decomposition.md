---
template:
  id: http://www.example.com/method/editors/system-decomposition
  name: "System Decomposition"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# System Decomposition

Create subsystems, add their contained physical parts, and record direct subsystem constraints. Enter part mass in kilograms when it is modeled.

```table-editor
---
columns: { this: { label: "Subsystem" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix method: <http://www.example.com/method/vocabulary#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

method:SubsystemShape
    a sh:NodeShape ;
    sh:targetClass method:Subsystem ;
    sh:property [
        sh:path method:contains ;
        sh:name "Contained parts" ;
        sh:class method:PhysicalPart ;
        dash:composite true ;
        oml:localReference true ;
    ] ;
    sh:property [
        sh:path method:constrains ;
        sh:name "Directly constrains" ;
        sh:class method:Subsystem ;
        oml:localReference true ;
    ] ;
    .

method:PhysicalPartShape
    a sh:NodeShape ;
    sh:targetClass method:PhysicalPart ;
    sh:targetNode <http://www.example.com/project/description#Part_NoseCone> ;
    sh:property [
        sh:path method:massKg ;
        sh:name "Mass (kg)" ;
        sh:datatype xsd:decimal ;
        sh:maxCount 1 ;
        sh:minCount 2 ;
    ] ;
    .
```
