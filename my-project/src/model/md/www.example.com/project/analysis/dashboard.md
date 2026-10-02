---
ontology: http://www.example.com/project/description.bundle#
---
# Project Analysis Dashboard

This page composes the reusable method dashboard against the hypersonic vehicle
description.

```compose
template: http://www.example.com/method/analysis/dashboard
```

For pattern-level orphan checks, see [Pattern Gap Queries](./gap-queries.md).

## Computed direct-tie analysis

This project-specific script queries the asserted constraint edges, computes
incoming and outgoing degree, ranks the subsystems, and renders the tie counts.
The computation counts both incoming and outgoing direct ties and reports
every subsystem tied for the maximum.

```python
result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?source ?target
WHERE { ?source method:constrains ?target }
ORDER BY ?source ?target
""")

from collections import Counter
from html import escape

incoming = Counter()
outgoing = Counter()
subsystems = set()
for row in result["rows"]:
    source = row["source"]
    target = row["target"]
    subsystems.update((source, target))
    outgoing[source] += 1
    incoming[target] += 1

def label(iri):
    return iri.rsplit("#", 1)[-1]

degrees = {
    subsystem: incoming[subsystem] + outgoing[subsystem]
    for subsystem in subsystems
}
highest = max(degrees.values(), default=0)
leaders = sorted(label(s) for s, degree in degrees.items() if degree == highest)

body = "".join(
    "<tr><td>{}</td><td>{}</td><td>{}</td><td>{}</td></tr>".format(
        escape(label(subsystem)),
        outgoing[subsystem],
        incoming[subsystem],
        degrees[subsystem],
    )
    for subsystem in sorted(degrees, key=lambda s: (-degrees[s], label(s)))
)
display(
    "<p>{} direct constraints; highest incident degree {}: {}.</p>".format(
        len(result["rows"]), highest, ", ".join(map(escape, leaders))
    )
    + "<table><thead><tr><th>Subsystem</th><th>Out</th><th>In</th>"
    + "<th>Total direct ties</th></tr></thead><tbody>"
    + body
    + "</tbody></table>"
)
```

## Rudimentary margin-based reliability estimate

The standard deviation value of `0.3` is illustrative. This script marks each
part as a pass when `marginOfSafety - standardDeviation >= 1`, then reports the
passing-part fraction. This is the requested screening metric, not a calibrated
probability of reliability.

```python
result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?part ?margin ?sigma
WHERE {
  ?part method:marginOfSafety ?margin ;
        method:standardDeviation ?sigma .
}
ORDER BY ?part
""")

from html import escape

parts = []
for row in result["rows"]:
    margin = float(row["margin"])
    sigma = float(row["sigma"])
    adjusted_margin = margin - sigma
    passed = adjusted_margin >= 1.0
    parts.append((row["part"], margin, sigma, adjusted_margin, passed))

if not parts:
    raise ValueError("Cannot calculate reliability score: no parts have both MOS and standard deviation.")

score = sum(1 for part in parts if part[4]) / len(parts)
rows_html = "".join(
    "<tr><td>{}</td><td>{:.2f}</td><td>{:.2f}</td><td>{:.2f}</td><td>{}</td></tr>".format(
        escape(part), margin, sigma, adjusted_margin, "Pass" if passed else "Fail"
    )
    for part, margin, sigma, adjusted_margin, passed in parts
)
display(
    "<p>Margin-based reliability score: {:.4f} ({}/{} parts pass).</p>".format(
        score, sum(1 for part in parts if part[4]), len(parts)
    )
    + "<table><thead><tr><th>Part</th><th>MOS</th><th>Std. dev.</th>"
    + "<th>MOS − σ</th><th>Result</th></tr></thead><tbody>"
    + rows_html
    + "</tbody></table>"
)
```

## Illustrative powered-flight range estimate

The model uses the Breguet jet-range relation
`R = (V / (g · TSFC)) · (L/D) · ln(Winitial / Wfinal)`, with mass ratios used
in place of weight ratios. Here TSFC is in `kg/(N·s)`, cruise speed is in
`m/s`, and `g = 9.80665 m/s²`. Thermal-stackup displacement is an illustrative
equivalent fuel-capacity estimate; thermal-stackup mass is also included in dry
mass. Actual fuel used is capped by both the modeled fuel load and the remaining
fuel capacity. This powered-cruise form uses cruise speed directly, so it does
not require density or altitude; the separate glider relation `R = h × (L/D)`
does require altitude and is not used here.

```python
from math import log
from html import escape

dry_mass_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?mass
WHERE {
  ?part method:massKg ?mass .
  FILTER(?part != <http://www.example.com/project/description#Part_Fuel>)
}
""")
if not dry_mass_result["rows"]:
    raise ValueError("Cannot calculate range: no non-fuel part masses are modeled.")
dry_mass_kg = sum(float(row["mass"]) for row in dry_mass_result["rows"])

fuel_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?fuelLoad ?fuelCapacity
WHERE {
  <http://www.example.com/project/description#Part_Fuel>
    method:massKg ?fuelLoad ;
    method:fuelCapacityKg ?fuelCapacity .
}
""")
if len(fuel_result["rows"]) != 1:
    raise ValueError("Cannot calculate range: expected one fuel load and capacity.")
fuel_load_kg = float(fuel_result["rows"][0]["fuelLoad"])
nominal_fuel_capacity_kg = float(fuel_result["rows"][0]["fuelCapacity"])

displacement_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?displacement
WHERE { ?stackup method:fuelCapacityDisplacementKg ?displacement }
""")
thermal_displacement_kg = sum(
    float(row["displacement"]) for row in displacement_result["rows"]
)
available_capacity_kg = max(
    0.0, nominal_fuel_capacity_kg - thermal_displacement_kg
)
fuel_used_kg = min(fuel_load_kg, available_capacity_kg)
if dry_mass_kg <= 0 or fuel_used_kg <= 0:
    raise ValueError("Cannot calculate range: dry mass and usable fuel must be positive.")

ld_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?ld WHERE {
  <http://www.example.com/project/description#Part_Wings>
    method:liftToDragRatio ?ld
}
""")
speed_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?speed WHERE {
  <http://www.example.com/project/description#Sys_Propulsion>
    method:cruiseSpeedMps ?speed
}
""")
tsfc_result = await query("""
PREFIX method: <http://www.example.com/method/vocabulary#>
SELECT ?tsfc WHERE {
  <http://www.example.com/project/description#Part_Engine>
    method:thrustSpecificFuelConsumption ?tsfc
}
""")
for name, result in (
    ("wing lift-to-drag ratio", ld_result),
    ("cruise speed", speed_result),
    ("engine TSFC", tsfc_result),
):
    if len(result["rows"]) != 1:
        raise ValueError("Cannot calculate range: expected exactly one {} estimate.".format(name))

ld_ratio = float(ld_result["rows"][0]["ld"])
cruise_speed_mps = float(speed_result["rows"][0]["speed"])
tsfc_kg_per_newton_second = float(tsfc_result["rows"][0]["tsfc"])
if ld_ratio <= 0 or cruise_speed_mps <= 0 or tsfc_kg_per_newton_second <= 0:
    raise ValueError("Cannot calculate range: L/D, speed, and TSFC must be positive.")

initial_mass_kg = dry_mass_kg + fuel_used_kg
final_mass_kg = dry_mass_kg
range_km = (
    cruise_speed_mps
    / (9.80665 * tsfc_kg_per_newton_second)
    * ld_ratio
    * log(initial_mass_kg / final_mass_kg)
    / 1000.0
)
display(
    "<p>Illustrative powered-flight range: {:.1f} km.</p>"
    "<ul><li>Dry mass: {:.1f} kg</li>"
    "<li>Wing L/D: {:.2f}</li>"
    "<li>Cruise speed: {:.1f} m/s</li>"
    "<li>TSFC: {:.6g} kg/(N·s)</li>"
    "<li>Nominal / available fuel capacity: {:.1f} / {:.1f} kg</li>"
    "<li>Fuel load / usable fuel: {:.1f} / {:.1f} kg</li>"
    "<li>Thermal-stackup capacity displacement: {:.1f} kg-equivalent</li></ul>".format(
        range_km,
        dry_mass_kg,
        ld_ratio,
        cruise_speed_mps,
        tsfc_kg_per_newton_second,
        nominal_fuel_capacity_kg,
        available_capacity_kg,
        fuel_load_kg,
        fuel_used_kg,
        thermal_displacement_kg,
    )
)
```
